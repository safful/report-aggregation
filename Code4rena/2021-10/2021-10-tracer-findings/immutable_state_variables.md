## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Immutable state variables](https://github.com/code-423n4/2021-10-tracer-findings/issues/32) 

# Handle

pauliax


# Vulnerability details

## Impact
There are variables that do not change so they can be marked as immutable to greatly improve the gast costs. Examples of such variables are: scaler and oracle in ChainlinkOracleWrapper, factory in PoolCommitterDeployer, poolName in LeveragedPool and there are many more.

## Recommended Mitigation Steps
Consider applying 'immutable' to reduce gas costs.


