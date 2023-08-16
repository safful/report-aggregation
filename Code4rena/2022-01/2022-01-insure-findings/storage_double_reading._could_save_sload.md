## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Storage double reading. Could save SLOAD](https://github.com/code-423n4/2022-01-insure-findings/issues/5) 

# Handle

robee


# Vulnerability details

Reading a storage variable is gas costly (SLOAD). In cases of multiple read of a storage variable in the same scope, caching the first read (i.e saving as a local variable) can save gas and decrease the
 overall gas uses. The following is a list of functions and the storage variables that you read twice: 

        CDSTemplate.sol: parameters.getLockup is read twice in withdraw
        Factory.sol: registry is read twice in createMarket
        IndexTemplate.sol: totalAllocPoint is read twice in set
        IndexTemplate.sol: MAGIC_SCALE_1E6 is read twice in withdrawable
        PoolTemplate.sol: parameters.getLockup is read twice in withdraw
        PoolTemplate.sol: lockedAmount is read twice in utilizationRate
        PoolTemplate.sol: MAGIC_SCALE_1E6 is read twice in allocateCredit
        PoolTemplate.sol: MAGIC_SCALE_1E6 is read twice in withdrawCredit
        PoolTemplate.sol: MAGIC_SCALE_1E6 is read twice in insure
        PoolTemplate.sol: MAGIC_SCALE_1E6 is read twice in resume
        BondingPremium.sol: k is read twice in getCurrentPremiumRate
        BondingPremium.sol: k is read twice in getPremiumRate
        BondingPremium.sol: c is read twice in getPremiumRate
        BondingPremium.sol: b is read twice in getCurrentPremiumRate
        BondingPremium.sol: b is read twice in getPremiumRate
        BondingPremium.sol: T_1 is read twice in getCurrentPremiumRate
        BondingPremium.sol: T_1 is read twice in getPremiumRate
        BondingPremium.sol: BASE is read twice in getCurrentPremiumRate
        BondingPremium.sol: BASE is read twice in getPremiumRate
        BondingPremium.sol: BASE_x2 is read twice in getCurrentPremiumRate
        BondingPremium.sol: BASE_x2 is read twice in getPremiumRate
        Vault.sol: token is read twice in repayDebt
        Vault.sol: token is read twice in utilize
        Vault.sol: token is read twice in withdrawRedundant
        Vault.sol: totalAttributions is read twice in attributionValue
        Vault.sol: balance is read twice in valueAll
        Vault.sol: balance is read twice in withdrawRedundant


