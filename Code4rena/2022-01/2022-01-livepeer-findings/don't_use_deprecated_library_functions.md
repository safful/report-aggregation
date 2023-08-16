## Tags

- bug
- 0 (Non-critical)
- resolved
- sponsor confirmed

# [Don't use deprecated library functions](https://github.com/code-423n4/2022-01-livepeer-findings/issues/207) 

# Handle

byterocket


# Vulnerability details

## Impact

The `_setupRole` function in OpenZeppelin's `AccessControl` contract is marked
as deprecated in favor of `_grantRole`.
See [here](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/AccessControl.sol#L183).

Following contracts use the deprecated `_setupRole` in their constructor:
```
arbitrum-lpt-bridge:
  - ControlledGateway.sol
  - L1/escrow/L1Escrow.sol
  - L2/gateway/L2Migrator.sol
  - token/LivepeerToken.sol
  - L1/gateway/L1Migrator.sol
```

## Recommended Mitigation Steps

Refactor the contracts constructor's to use `_grantRole` instead of `_setupRole`.

