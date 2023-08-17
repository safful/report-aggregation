## Tags

- bug
- question
- 3 (High Risk)
- primary issue
- sponsor confirmed
- syncRewards wrong nextRewards

# [Wrong accounting logic when syncRewards() is called within beforeWithdraw makes withdrawals impossible](https://github.com/code-423n4/2022-09-frax-findings/issues/15) 

# Lines of code

https://github.com/code-423n4/2022-09-frax/blob/dc6684f77b4e9bd965e8862be7f5fb71473a4c4c/src/sfrxETH.sol#L50


# Vulnerability details

## Impact
`sfrxETH.beforeWithdraw` first calls the `beforeWithdraw` of `xERC4626`, which decrements `storedTotalAssets` by the given amount. If the timestamp is greater than the `rewardsCycleEnd`, `syncRewards` is called. However, the problem is that the assets have not been transfered out yet, meaning `asset.balanceOf(address(this))` still has the old value. On the other hand, `storedTotalAssets` was already updated. Therefore, the following calculation will be inflated by the amount for which the withdrawal was requested:
```
uint256 nextRewards = asset.balanceOf(address(this)) - storedTotalAssets_ - lastRewardAmount_;
```
This has severe consequences:
1.) During the following reward period, `lastRewardAmount` is too high, which means that too much rewards are paid out too users who want to withdraw. A user could exploit this to steal the assets of other users.
2.) When `syncRewards()` is called the next time, it is possible that the `nextRewards` calculation underflows because `lastRewardAmount > asset.balanceOf(address(this))`. This is very bad because `syncRewards()` will be called in every withdrawal (after the `rewardsCycleEnd`) and none of them will succeed because of the underflow. Depositing more also does not help here, it just increases `asset.balanceOf(address(this))` and `storedTotalAssets` by the same amount, which does not eliminate the underflow.

Note that this bug does not require a malicious user or a targeted attack to surface. It can (and probably will) happen in practice just by normal user interactions with the vault (which is for instance shown in the PoC).

## Proof Of Concept
Consider the following test:
```
function testTotalAssetsAfterWithdraw() public {        
        uint128 deposit = 1 ether;
        uint128 withdraw = 1 ether;
        // Mint frxETH to this testing contract from nothing, for testing
        mintTo(address(this), deposit);

        // Generate some sfrxETH to this testing contract using frxETH
        frxETHtoken.approve(address(sfrxETHtoken), deposit);
        sfrxETHtoken.deposit(deposit, address(this));
        require(sfrxETHtoken.totalAssets() == deposit);

        vm.warp(block.timestamp + 1000);
        // Withdraw frxETH (from sfrxETH) to this testing contract
        sfrxETHtoken.withdraw(withdraw, address(this), address(this));
        vm.warp(block.timestamp + 1000);
        sfrxETHtoken.syncRewards();
        require(sfrxETHtoken.totalAssets() == deposit - withdraw);
    }
```

This is a normal user interaction where a user deposits into the vault, and makes a withdrawal some time later. However, at this point the `syncRewards()` within the `beforeWithdraw` is executed. Because of that, the documented accounting mistake happens and the next call (in fact every call that will be done in the future) to `syncRewards()` reverts with an underflow.

## Recommended Mitigation Steps
Call `syncRewards()` before decrementing `storedTotalAssets`, i.e.:
```
function beforeWithdraw(uint256 assets, uint256 shares) internal override {
	if (block.timestamp >= rewardsCycleEnd) { syncRewards(); }
	super.beforeWithdraw(assets, shares); // call xERC4626's beforeWithdraw AFTER
}
```
Then, `asset.balanceOf(address(this))` and `storedTotalAssets` are still in sync within `syncRewards()`.