## Tags

- bug
- 0 (Non-critical)
- resolved
- sponsor confirmed

# [Misleading comments and documentation](https://github.com/code-423n4/2022-01-insure-findings/issues/337) 

# Handle

pauliax


# Vulnerability details

## Impact
There are some issues with comments/documentation, e.g.:
Misleading comment:
```solidity
   * @return true if the id within the market already exists
  function getCDS(address _address) external view override returns (address)
```
No such function (present in documentation):
```solidity
  function getInsuranceCount(address _user)
```
"getInsuranceCount returns how many insurance policies the specified user has."

## Recommended Mitigation Steps
Consider revisiting and updating discrepancies between the documentation and comments.

