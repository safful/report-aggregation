## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed
- ATokenYieldSource

# [_depositToAave always returns 0](https://github.com/code-423n4/2021-06-pooltogether-findings/issues/80) 

# Handle

pauliax


# Vulnerability details

## Impact
contract ATokenYieldSource function _depositToAave returns 0 if successful. However, this value is not checked nor used anywhere. As this function is internal it would probably be better to remove this unnecessary return to save some gas and eliminate confusion.

## Recommended Mitigation Steps
refactor function _depositToAave to return void.

