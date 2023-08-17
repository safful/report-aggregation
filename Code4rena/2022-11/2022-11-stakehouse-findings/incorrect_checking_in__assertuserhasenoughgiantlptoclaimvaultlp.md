## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-25

# [Incorrect checking in _assertUserHasEnoughGiantLPToClaimVaultLP](https://github.com/code-423n4/2022-11-stakehouse-findings/issues/382) 

# Lines of code

https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantPoolBase.sol#L93-L97


# Vulnerability details

## Impact
The batch operations of `withdrawDETH()` in GiantSavETHVaultPool.sol and `withdrawLPTokens()` in GiantPoolBase.sol are meaningless because they will fail whenever more than one lpToken is passed.
Each user can perform `withdrawDETH()` or `withdrawLPTokens()` with one LPToken only once a day.

## Proof of Concept

Both the `withdrawDETH()` in GiantSavETHVaultPool.sol and `withdrawLPTokens()` in GiantPoolBase.sol will call `GiantPoolBase._assertUserHasEnoughGiantLPToClaimVaultLP(lpToken, amount)` and `lpTokenETH.burn(msg.sender, amount)`:

There is a require in `_assertUserHasEnoughGiantLPToClaimVaultLP()`:
```
require(lpTokenETH.lastInteractedTimestamp(msg.sender) + 1 days < block.timestamp, "Too new");
```

At the same time, `lpTokenETH.burn(msg.sender, amount)` will update `lastInteractedTimestamp[msg.sender]` to latest block timestamp in `_afterTokenTransfer()` of GiantLP.sol.

So, a user can perform `withdrawDETH` or `withdrawLPTokens` of one LPToken only once a day, others more will fail by `_assertUserHasEnoughGiantLPToClaimVaultLP()`.

## Tools Used
VS Code

## Recommended Mitigation Steps

The LPToken being operated on should be checked for lastInteractedTimestamp rather than lpTokenETH.

```
diff --git a/contracts/liquid-staking/GiantPoolBase.sol b/contracts/liquid-staking/GiantPoolBase.sol
index 8a8ff70..5c009d9 100644
--- a/contracts/liquid-staking/GiantPoolBase.sol
+++ b/contracts/liquid-staking/GiantPoolBase.sol
@@ -93,7 +93,7 @@ contract GiantPoolBase is ReentrancyGuard {
     function _assertUserHasEnoughGiantLPToClaimVaultLP(LPToken _token, uint256 _amount) internal view {
         require(_amount >= MIN_STAKING_AMOUNT, "Invalid amount");
         require(_token.balanceOf(address(this)) >= _amount, "Pool does not own specified LP");
-        require(lpTokenETH.lastInteractedTimestamp(msg.sender) + 1 days < block.timestamp, "Too new");
+        require(_token.lastInteractedTimestamp(msg.sender) + 1 days < block.timestamp, "Too new");
     }

     /// @dev Allow an inheriting contract to have a hook for performing operations post depositing ETH
```

