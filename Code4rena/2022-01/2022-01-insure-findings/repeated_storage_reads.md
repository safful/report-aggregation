## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Repeated storage reads](https://github.com/code-423n4/2022-01-insure-findings/issues/306) 

# Handle

pauliax


# Vulnerability details

## Impact
Repeated storage read should be cached, e.g.
attributions[_target] is read from storage twice:
```solidity
        if (attributions[_target] > 0) {
            return (valueAll() * attributions[_target]) / totalAttributions;
```
totalAttributions read twice:
```solidity
        if (totalAttributions > 0 && _attribution > 0) {
            return (_attribution * valueAll()) / totalAttributions;
```
available() called twice:
```solidity
        if (available() < _retVal) {
            uint256 _shortage = _retVal - available();
```
would be cheaper to use _token from memory here:
```solidity
    IERC20(token).safeTransfer(_to, _redundant);
```

There are more places where this optimization could be applied besides the provided examples, but the basic idea is to cache storage variables if you need to access them multiple times when the value does not change.

