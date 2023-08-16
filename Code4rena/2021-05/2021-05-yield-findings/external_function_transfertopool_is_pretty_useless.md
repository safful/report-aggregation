## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [external function transferToPool is pretty useless](https://github.com/code-423n4/2021-05-yield-findings/issues/57) 

# Handle

pauliax


# Vulnerability details

## Impact
external function transferToPool is pretty useless and error-prone. It relies on the user not to leave these tokens in a separate tx, otherwise, it will just be feeding the bots. To use it directly users will have to write their own custom smart contract and chain actions.

## Recommended Mitigation Steps
It would be better to remove this function and leave the only way to invoke it via a batch function.

