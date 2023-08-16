## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Use immutable keyword](https://github.com/code-423n4/2021-06-pooltogether-findings/issues/7) 

# Handle

gpersoon


# Vulnerability details

## Impact
Most of the contracts set variables in the initialize function that are never changed. See for examples in the "proof of concept" section.
Here the solidity keyword "immutable" could be added to the variables as an extra security measure.

## Proof of Concept
ControlledToken.sol:         TokenControllerInterface public override controller;
StakePricePools.sol:           IERC20Upgradeable private stakeToken;
YieldSourcePrizePool.sol:   IYieldSource public yieldSource;
PrizePool.sol:                     RegistryInterface public reserveRegistry;
PrizePool.sol:                     uint256 public maxExitFeeMantissa;
PrizePool.sol:                     uint256 public maxTimelockDuration;
BadgerYieldSource.sol       IBadgerSett private immutable badgerSett;
BadgerYieldSource.sol       IBadger private immutable badger;
YearnV2YieldSource.sol      IYVaultV2 public vault;
YearnV2YieldSource.sol      IERC20Upgradeable internal token; 
IdleYieldSource.sol            address public idleToken;
IdleYieldSource.sol            address public underlyingAsset;
ATokenYieldSource.sol       ATokenInterface public aToken;
ATokenYieldSource.sol       ILendingPoolAddressesProviderRegistry public lendingPoolAddressesProviderRegistry;    
SushiYieldSource.sol          ISushiBar public immutable sushiBar;
SushiYieldSource.sol          ISushi public immutable sushiAddr;   

## Tools Used

## Recommended Mitigation Steps
Add immutable where possible

