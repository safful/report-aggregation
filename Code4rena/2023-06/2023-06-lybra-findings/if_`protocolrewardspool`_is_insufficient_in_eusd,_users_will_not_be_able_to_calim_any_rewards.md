## Tags

- bug
- 2 (Med Risk)
- downgraded by judge
- primary issue
- satisfactory
- selected for report
- sponsor acknowledged
- edited-by-warden
- M-17

# [If `ProtocolRewardsPool` is insufficient in EUSD, users will not be able to calim any rewards](https://github.com/code-423n4/2023-06-lybra-findings/issues/223) 

# Lines of code

https://github.com/code-423n4/2023-06-lybra/blob/main/contracts/lybra/miner/ProtocolRewardsPool.sol#L196


# Vulnerability details

## Impact
If [`ProtocolRewardsPool`](https://github.com/code-423n4/2023-06-lybra/blob/main/contracts/lybra/miner/ProtocolRewardsPool.sol) is insufficient in EUSD, but has enough PeUSD to give rewards it still reverts, due to wrong `if()` statement, thus it is  unable to send the rewards to users.

## Proof of Concept
Users have just emptied [`ProtocolRewardsPool`](https://github.com/code-423n4/2023-06-lybra/blob/main/contracts/lybra/miner/ProtocolRewardsPool.sol) out of EUSD, by claiming rewards with  `getReward`. Now the protocol has a new distribution of PeUSD tokens, with `LybraConfigurator.distributeRewards`, but when users try to claim their rewards [`getReward`](https://github.com/code-423n4/2023-06-lybra/blob/main/contracts/lybra/miner/ProtocolRewardsPool.sol#L190-L218) reverts because of this:
```jsx
   function getReward() external updateReward(msg.sender) {
        uint reward = rewards[msg.sender];
        if (reward > 0) {
            rewards[msg.sender] = 0;
            IEUSD EUSD = IEUSD(configurator.getEUSDAddress());//get the address
            uint256 balance = EUSD.sharesOf(address(this));//get the balance == 
//@aduit here eUSDShare = balance >= reward-false => reward - balance => rewards - 0 | eUSDShare = reward
            uint256 eUSDShare = balance >= reward ? reward : reward - balance;
//here it tries to send the rewards amount, but it reverts since it has not tokens 
            EUSD.transferShares(msg.sender, eUSDShare);

```
Because of the constant revert users are not able to claim their rewards and need to wait for EUSD distribution. The other bad thing is that the PeUSD is uncalimable to most extent.Again because of the line bellow, if:

 - Protocol has 40e18 EUSD and 100e18 PeUSD.
 - UserA tries to claim his rewards, that are 100e18 in rewards tokens.
```jsx
//eUSDShare = balance >= reward-false => reward - balance => 100e18 - 40e18 => eUSDShare = 60e18 
uint256 eUSDShare = balance >= reward ? reward : reward - balance;
//again reverts, because contract has 40, whily trying to send 60
EUSD.transferShares(msg.sender, eUSDShare);
```
Now PeUSD is un-claimable and remains in the contract.
### Foundry PoC
```jsx
    function test_no_EUSD() public {
        //make 2 random users
        deal(address(lbr), user1, 1000e18);
        deal(address(lbr), user2, 4000e18);

        //stake for bolt of them
        vm.prank(user1);
        rewardsPool.stake(1000e18); 

        vm.prank(user2);
        rewardsPool.stake(4000e18);   

        //get some PeUSD in the config and call distributeRewards() to send it to the pool
        //@notice here we don't send any EUSD => rewardsPool has 0 EUSD
        deal(address(PeUSD),address(configurator),1e21);
        configurator.distributeRewards();

        //to make sure the balance is sent
        PeUSD.balanceOf(address(rewardsPool));

        //user rewards is actually 2e17 per 1e18 => 2e20 total for user1
        vm.prank(user1);
        //but here reverts, because it is unable to send any EUSD
        rewardsPool.getReward();
        console.log(rewardsPool.earned(user1));
        console.log("pEUSD user1: ", PeUSD.balanceOf(user1));
        console.log("pEUSD pool : ", PeUSD.balanceOf(address(rewardsPool)));
        console.log();
    }
```
## Tools Used
Manual Review

## Recommended Mitigation Steps
update the [`if`](https://github.com/code-423n4/2023-06-lybra/blob/main/contracts/lybra/miner/ProtocolRewardsPool.sol) as:
```jsx
-  uint256 eUSDShare = balance >= reward ? reward : reward - balance;
+  uint256 eUSDShare = balance >= reward ? reward : balance;
```





## Assessed type

Math