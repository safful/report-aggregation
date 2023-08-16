## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [`withdrawSeq` might not be set](https://github.com/code-423n4/2021-10-union-findings/issues/62) 

# Handle

cmichel


# Vulnerability details

The `AssetManager.withdraw` function iterates through the markets based on the `withdrawSeq` array field.
This field must be manually set to cover all markets on each new market addition.

## Impact
It could be that a market is added but this array is not updated.
Thus not all markets are iterated and users might not be able to withdraw their entire `amount` as the new market is skipped

## Recommended Mitigation Steps
Ensure that `withdrawSeq` is always up-to-date when `addAdapter` is called, for example, `addAdapter` could add the new adapter as the last element to `withdrawSeq` until it's manually set through `changeWithdrawSequence`.


