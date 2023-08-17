## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed
- reviewed

# [`getNewCurrentFees` reverts when `minFeePercentage` > `feeRatio`](https://github.com/code-423n4/2022-04-backd-findings/issues/50) 

# Lines of code

https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/contracts/pool/LiquidityPool.sol#L694


# Vulnerability details

## Impact
Depositors won't be able to transfer or redeem funds temporarily.


The problem is caused by the implementation of `LiquidityPool.getNewCurrentFees`:

```
function getNewCurrentFees(
    uint256 timeToWait,
    uint256 lastActionTimestamp,
    uint256 feeRatio
) public view returns (uint256) {
    uint256 timeElapsed = _getTime() - lastActionTimestamp;
    uint256 minFeePercentage = getMinWithdrawalFee();
    if (timeElapsed >= timeToWait) {
        return minFeePercentage;
    }
    uint256 elapsedShare = timeElapsed.scaledDiv(timeToWait);
    return feeRatio - (feeRatio - minFeePercentage).scaledMul(elapsedShare);
}
```
The last line requires the current `feeRatio` to be higher than `minFeePercentage` or the function will revert. When this condition is broken, some critical functions such as transferring tokens and redeeming will be unusable. Affected users need to wait until enough time has elapsed and `getNewCurrentFees` returns `minFeePercentage` on [L691](https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/contracts/pool/LiquidityPool.sol#L691).

This could happen if governance changes the `MinWithdrawalFee` to be higher than a user's feeRatio.

## Proof of Concept
- Initial `MinWithdrawalFee` is set to 0, `MaxWithdrawalFee` is set to 0.03e18.
- Alice deposits fund and receives LP token. Alice's `feeRatio` is now set to 0.03e18 (the current `MaxWithdrawalFee`).
- Governance changes `MaxWithdrawalFee` to `0.05e18` and `MinWithdrawalFee` to `0.04e18`.
- `minFeePercentage` is now higher than Alice's `feeRatio` and she can't transfer nor redeem the LP token until `timeElapsed >= timeToWait`.

## Recommended Mitigation Steps
Add a new condition in `getNewCurrentFees` [L690](https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/contracts/pool/LiquidityPool.sol#L690) to account for this case:
```
if (timeElapsed >= timeToWait || minFeePercentage > feeRatio) {
    return minFeePercentage;
}
```


