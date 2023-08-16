## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [No sanity check of safe ratio when adding collateral](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/217) 

# Handle

kenzo


# Vulnerability details

When changing collateral's ratio, it is rightly checked to be smaller than 110%.
However when adding new collateral, the ratio check is not there, so it can be added with ratio that is larger than 110%.

## Impact
Accidentally adding an asset with larger ratio would result in users being able to withdraw more YUSD than supplied VC.

## Proof of Concept
When an asset is being added, there is no sanity check that the ratio is within the correct range. [(Code ref)](https://github.com/code-423n4/2021-12-yetifinance/blob/main/packages/contracts/contracts/Dependencies/Whitelist.sol#L92:#L127)

This is unlike `changeRatio`, which validates that the new ratio is in correct range. [(Code ref)](https://github.com/code-423n4/2021-12-yetifinance/blob/main/packages/contracts/contracts/Dependencies/Whitelist.sol#L204)
```
require(_ratio < 1100000000000000000, "ratio must be less than 1.10 => greater than 1.1 would mean taking out more YUSD than collateral VC");
```

## Recommended Mitigation Steps
Add the same ratio check to `addCollateral`.

