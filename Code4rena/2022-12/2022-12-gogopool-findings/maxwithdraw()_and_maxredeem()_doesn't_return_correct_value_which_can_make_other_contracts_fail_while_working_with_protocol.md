## Tags

- bug
- 2 (Med Risk)
- selected for report
- sponsor confirmed
- fix token (sponsor)
- M-16

# [maxWithdraw() and maxRedeem() doesn't return correct value which can make other contracts fail while working with protocol](https://github.com/code-423n4/2022-12-gogopool-findings/issues/476) 

# Lines of code

https://github.com/code-423n4/2022-12-gogopool/blob/aec9928d8bdce8a5a4efe45f54c39d4fc7313731/contracts/contract/tokens/TokenggAVAX.sol#L215-L223
https://github.com/code-423n4/2022-12-gogopool/blob/aec9928d8bdce8a5a4efe45f54c39d4fc7313731/contracts/contract/tokens/TokenggAVAX.sol#L205-L213


# Vulnerability details

## Impact
Functions `maxWithdraw()` and `maxRedeem()` returns max amount of assets or shares owner would be able to withdraw taking into account liquidity in the TokenggAVAX contract, but logics don't consider that when user withdraws the withdrawal amounts subtracted from `totalReleasedAssets` (in `beforeWithdraw()` function) so the maximum amounts that can user withdraws should always be lower than `totalReleasedAssets` (which shows all the deposits and withdraws) but because functions `maxWithdraw()` and `maxRedeem()` uses `totalAssets()` to calculate available AVAX which includes deposits and current cycle rewards so those functions would return wrong value (whenever the return value is bigger than `totalReleaseAssets` then it would be wrong)

## Proof of Concept
This is `beforeWithdraw()` code:
```
	function beforeWithdraw(
		uint256 amount,
		uint256 /* shares */
	) internal override {
		totalReleasedAssets -= amount;
	}
```
This is `beforeWithdraw()` code which is called whenever users withdraws their funds and as you can see the amount of withdrawal assets subtracted from `totalReleaseAssets` so withdrawal amounts can never be bigger than `totalReleaseAssets`. This is `maxWithdraw()` code:
```
	function maxWithdraw(address _owner) public view override returns (uint256) {
		if (getBool(keccak256(abi.encodePacked("contract.paused", "TokenggAVAX")))) {
			return 0;
		}
		uint256 assets = convertToAssets(balanceOf[_owner]);
		uint256 avail = totalAssets() - stakingTotalAssets;
		return assets > avail ? avail : assets;
	}
```
As you can see to calculate available AVAX in the contract address code uses `totalAssets() - stakingTotalAssets` and `totalAssets()` shows deposits + current cycle rewards so `totalAssets()` is bigger than `totalReleaseAssets` and the value of the `totalAssets() - stakingTotalAssets` can be bigger than `totalReleaseAssets` and if code returns `avail` as answer then the return value would be wrong.
imagine this scenario:
1. `totalReleaseAssets` is `10000` AVAX.
2. `stakingTotalAssets` is `1000` AVAX.
3. current cycle rewards is `4000` AVAX and `block.timestamp` is currently in the middle of the cycle so current rewards is `2000` AVAX.
4. `totalAssets()` is `totalReleaseAssets + current rewards = 10000 + 2000 = 12000`. 
5. contract balance is `10000 + 4000 - 1000 = 13000` AVAX.
6. user1 has 90% contract shares and calls `maxWithdraw()` and code would calculate user assets as `10800` AVAX and available AVAX in contract as `totalAssets() - stakingTotalAssets = 12000 - 1000 = 11000` and code would return `10800` as answer.
7. now if user1 withdraws `10800` AVAX code would revert in the function `beforeWithdraw()` because code would try to execute `totalReleaseAssets = totalReleaseAssets - amount = 10000 - 10800` and it would revert because of the underflow. so in reality user1 couldn't withdraw `10800` AVAX which was the return value of the `maxWithdraw()` for user1.

the root cause of the bug is that the withdrawal amount is subtracted from `totalReleaseAssets` and so max withdrawal can never be `totalReleaseAssets` and function `maxWithdraw()` should never return value bigger than `totalReleaseAssets`. (the bug in function `maxRedeem()` is similar)

This bug would cause other contract or front end calls to fail, for example if the logic is something like this:
```
   amount = maxWithdraw(user);
   TokenggAVAX.withdrawAVAX(amount);
```
according the function definitions this code should work bug because of the the issue there are situations that this would revert and other contracts and UI can't work properly with the protocol.

## Tools Used
VIM

## Recommended Mitigation Steps
consider `totalReleaseAssets` in max withdrawal amount too.