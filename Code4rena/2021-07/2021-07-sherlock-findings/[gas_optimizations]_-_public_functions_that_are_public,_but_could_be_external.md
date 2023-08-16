## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [[Gas optimizations] - Public functions that are public, but could be external](https://github.com/code-423n4/2021-07-sherlock-findings/issues/112) 

# Handle

a_delamo


# Vulnerability details

## Impact

The following functions are public, but they could be decla

```
getUnactivatedStakersPoolBalance(IERC20) should be declared external:
        - PoolBase.getUnactivatedStakersPoolBalance(IERC20) (contracts/facets/PoolBase.sol#146-148)
getTotalUnmintedSherX(IERC20) should be declared external:
        - PoolBase.getTotalUnmintedSherX(IERC20) (contracts/facets/PoolBase.sol#170-173)
accruedDebt(bytes32,IERC20) should be declared external:
        - LibPool.accruedDebt(bytes32,IERC20) (contracts/libraries/LibPool.sol#31-34)
getTotalAccruedDebt(IERC20) should be declared external:
        - LibPool.getTotalAccruedDebt(IERC20) (contracts/libraries/LibPool.sol#36-39)
accrueSherX(IERC20) should be declared external:
        - LibSherX.accrueSherX(IERC20) (contracts/libraries/LibSherX.sol#75-81)
accrueSherXWatsons() should be declared external:
        - LibSherX.accrueSherXWatsons() (contracts/libraries/LibSherX.sol#83-86)
deposit() should be declared external:
        - AaveV2.deposit() (contracts/strategies/AaveV2.sol#75-81)
```


## Tools Used

Slither 



