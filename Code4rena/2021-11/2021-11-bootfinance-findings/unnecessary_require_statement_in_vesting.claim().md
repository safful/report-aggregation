## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Unnecessary require statement in vesting.claim()](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/115) 

# Handle

Reigada


# Vulnerability details

## Impact
There is an unnecessary require statement in vesting.claim() -> https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/Vesting.sol#L197

This check is already done in https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/Vesting.sol#L186

## Tools Used
Manual testing

## Recommended Mitigation Steps
Remove the require statement in the claim() function as it is totally unnecessary. The check is already performed in the function _claimableAmount(address _addr).

