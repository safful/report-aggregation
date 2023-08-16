## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Functions should be written in mixedCase](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/163) 

# Handle

pmerkleplant


# Vulnerability details

Functions should be written in mixedCase, see [Solidity naming conventions](https://docs.soliditylang.org/en/v0.4.25/style-guide.html#function-names).

Functions breaking this convention:

- Function `_available_supply` in `vesting/contracts/AidropDistribution.sol`
- Function `available_supply` in `vesting/contracts/AidropDistribution.sol`
- Function `_available_supply` in `vesting/contracts/InvestorDistribution.sol`
- Function `available_supply` in `vesting/contracts/InvestorDistribution.sol`
- Function `dev_rugpull` in `vesting/contracts/InvestorDistribution.sol`

