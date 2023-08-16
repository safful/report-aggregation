## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [The address of Aave lendingPool may change](https://github.com/code-423n4/2021-08-floatcapital-findings/issues/99) 

# Handle

pauliax


# Vulnerability details

## Impact
contract YieldManagerAave caches lendingPool, however, in theory, it is possible that the implementation may change (see https://github.com/aave/aave-protocol/blob/4b4545fb583fd4f400507b10f3c3114f45b8a037/contracts/configuration/LendingPoolAddressesProvider.sol#L58-L65). I am not sure how likely in practice is that but a common solution that I see in other protocols that integrate with Aave is querying the lendingPool on the go (of course then you also need to handle the change in approvals).

## Recommended Mitigation Steps
An example solution you can see here: https://github.com/code-423n4/2021-07-sherlock/blob/d9c610d2c3e98a412164160a787566818debeae4/contracts/strategies/AaveV2.sol#L63-L65

