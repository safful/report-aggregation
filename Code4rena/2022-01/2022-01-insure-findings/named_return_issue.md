## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [Named return issue](https://github.com/code-423n4/2022-01-insure-findings/issues/18) 

# Handle

robee


# Vulnerability details

Users can mistakenly think that the return value is the named return, but it is actually the actualreturn statement that comes after. To know that the user needs to read the code and is confusing.
Furthermore, removing either the actual return or the named return will save gas. 

        CDSTemplate.sol, totalLiquidity
        Factory.sol, _createClone
        IndexTemplate.sol, withdrawable
        IndexTemplate.sol, leverage
        IndexTemplate.sol, totalLiquidity
        PoolTemplate.sol, availableBalance
        PoolTemplate.sol, utilizationRate
        PoolTemplate.sol, totalLiquidity

