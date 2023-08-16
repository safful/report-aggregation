## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Public functions to external](https://github.com/code-423n4/2022-01-insure-findings/issues/7) 

# Handle

robee


# Vulnerability details

The following functions could be set external to save gas and improve code quality. 
External call cost is less expensive than of public functions. 

        The function totalLiquidity in CDSTemplate.sol could be set external
        The function valueOfUnderlying in CDSTemplate.sol could be set external
        The function createMarket in Factory.sol could be set external
        The function totalLiquidity in IndexTemplate.sol could be set external
        The function set in IndexTemplate.sol could be set external
        The function leverage in IndexTemplate.sol could be set external
        The function withdrawable in IndexTemplate.sol could be set external
        The function valueOfUnderlying in IndexTemplate.sol could be set external
        The function deposit in IndexTemplate.sol could be set external
        The function adjustAlloc in IndexTemplate.sol could be set external
        The function allowance in InsureDAOERC20.sol could be set external
        The function decimals in InsureDAOERC20.sol could be set external
        The function totalSupply in InsureDAOERC20.sol could be set external
        The function name in InsureDAOERC20.sol could be set external
        The function transfer in InsureDAOERC20.sol could be set external
        The function increaseAllowance in InsureDAOERC20.sol could be set external
        The function transferFrom in InsureDAOERC20.sol could be set external
        The function decreaseAllowance in InsureDAOERC20.sol could be set external
        The function balanceOf in InsureDAOERC20.sol could be set external
        The function symbol in InsureDAOERC20.sol could be set external
        The function approve in InsureDAOERC20.sol could be set external
        The function getOwner in Parameters.sol could be set external
        The function utilizationRate in PoolTemplate.sol could be set external
        The function totalLiquidity in PoolTemplate.sol could be set external
        The function availableBalance in PoolTemplate.sol could be set external
        The function getPremium in PoolTemplate.sol could be set external
        The function valueOfUnderlying in PoolTemplate.sol could be set external
        The function unlock in PoolTemplate.sol could be set external
        The function originalLiquidity in PoolTemplate.sol could be set external
        The function deposit in PoolTemplate.sol could be set external
        The function allocatedCredit in PoolTemplate.sol could be set external
        The function getPremiumRate in BondingPremium.sol could be set external
        The function getCurrentPremiumRate in BondingPremium.sol could be set external
        The function getPricePerFullShare in Vault.sol could be set external
        The function valueAll in Vault.sol could be set external
        The function setController in Vault.sol could be set external
        The function underlyingValue in Vault.sol could be set external


