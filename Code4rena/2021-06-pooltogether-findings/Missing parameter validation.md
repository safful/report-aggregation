## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- ATokenYieldSource
- SushiYieldSource
- BadgerYieldSource
- PrizePool
- ControlledToken
- StakePrizePool

# [Missing parameter validation](https://github.com/code-423n4/2021-06-pooltogether-findings/issues/81) 

# Handle

cmichel


# Vulnerability details

Some parameters of functions are not checked for invalid values:
- `StakePrizePool.initialize`: `address _stakeToken` not checked for non-zero or contract
- `ControlledToken.initialize`: `address controller` not checked for non-zero or contract
- `PrizePool.withdrawReserve`: `address to` not checked for non-zero, funds will be lost when sending to zero address
- `ATokenYieldSource.initialize`: `address _aToken, _lendingPoolAddressesProviderRegistry` not checked for non-zero or contract
- `BadgerYieldSource.initialize`: `address badgerSettAddr, badgerAddr` not checked for non-zero or contract
- `SushiYieldSource.constructor`: `address _sushiBar, _sushiAddr` not checked for non-zero or contract

## Impact

Wrong user input or wallets defaulting to the zero addresses for a missing input can lead to the contract needing to redeploy or wasted gas.

## Recommended Mitigation Steps

Validate the parameters.

