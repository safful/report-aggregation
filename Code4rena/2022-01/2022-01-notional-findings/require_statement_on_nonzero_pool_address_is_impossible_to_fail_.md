## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Require statement on nonzero pool address is impossible to fail ](https://github.com/code-423n4/2022-01-notional-findings/issues/39) 

# Handle

TomFrenchBlockchain


# Vulnerability details

## Impact
Extra gas costs
## Proof of Concept

Here we check that `_noteETHPoolId` corresponds to a registered Balancer pool.

https://github.com/code-423n4/2022-01-notional/blob/d171cad9e86e0d02e0909eb66d4c24ab6ea6b982/contracts/TreasuryManager.sol#L59

`_balancerVault.getPool(_noteETHPoolId)` will revert if _noteETHPoolId is not a registered poolId

https://github.com/balancer-labs/balancer-v2-monorepo/blob/3c1c362adb1fa003cc33f64da93a2e286b5a1257/pkg/vault/contracts/PoolRegistry.sol#L93

https://github.com/balancer-labs/balancer-v2-monorepo/blob/3c1c362adb1fa003cc33f64da93a2e286b5a1257/pkg/vault/contracts/PoolRegistry.sol#L56

The require statement on the next line is impossible to fail unless we somehow manage to deploy a balancer pool to the zero address (which would be impressive)

https://github.com/code-423n4/2022-01-notional/blob/d171cad9e86e0d02e0909eb66d4c24ab6ea6b982/contracts/TreasuryManager.sol#L60

We can then safely remove this statement without any change in behaviour.

## Recommended Mitigation Steps

Remove require statement.

