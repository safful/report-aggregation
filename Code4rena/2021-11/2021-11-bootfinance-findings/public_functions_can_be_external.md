## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Public functions can be external](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/121) 

# Handle

nathaniel


# Vulnerability details

## Impact
Functions include `updateEmission(), dev_rugpull(), setAdmin(), revoke()
## Proof of Concept
https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/InvestorDistribution.sol#L185
https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/InvestorDistribution.sol#L203
https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/InvestorDistribution.sol#L212
https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/Vesting.sol#L104

