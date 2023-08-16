## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [A variable is being assigned its default value which is unnecessary.](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/87) 

# Handle

Jujic


# Vulnerability details

## Impact
Removing the assignment will save gas.
```
_totalSupply = 0;
```

## Proof of Concept
https://github.com/code-423n4/2021-12-yetifinance/blob/5f5bf61209b722ba568623d8446111b1ea5cb61c/packages/contracts/contracts/AssetWrappers/WJLP/WJLP.sol#L93

## Tools Used

## Recommended Mitigation Steps
Remove the assignment.

