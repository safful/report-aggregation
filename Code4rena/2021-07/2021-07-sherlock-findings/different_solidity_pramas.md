## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Different solidity pramas](https://github.com/code-423n4/2021-07-sherlock-findings/issues/16) 

# Handle

gpersoon


# Vulnerability details

## Impact
Several different solidity pragmas are uses for different solidity version.
Its cleaner to use the same version everywhere

## Proof of Concept
.\ForeignLock.sol:pragma solidity ^0.7.4;
.\NativeLock.sol:pragma solidity ^0.7.4;
.\facets\Gov.sol:pragma solidity ^0.7.4;
.\facets\GovDev.sol:pragma solidity ^0.7.0;
.\facets\Manager.sol:pragma solidity ^0.7.4;
.\facets\Payout.sol:pragma solidity ^0.7.4;
.\facets\PoolBase.sol:pragma solidity ^0.7.4;
.\facets\PoolDevOnly.sol:pragma solidity ^0.7.4;
.\facets\PoolOpen.sol:pragma solidity ^0.7.4;
.\facets\PoolStrategy.sol:pragma solidity ^0.7.4;
.\facets\SherX.sol:pragma solidity ^0.7.4;
.\facets\SherXERC20.sol:pragma solidity ^0.7.1;
.\interfaces\IGov.sol:pragma solidity ^0.7.4;
.\interfaces\IGovDev.sol:pragma solidity ^0.7.4;
.\interfaces\ILock.sol:pragma solidity ^0.7.4;
.\interfaces\IManager.sol:pragma solidity ^0.7.4;
.\interfaces\IPayout.sol:pragma solidity ^0.7.4;
.\interfaces\IPoolBase.sol:pragma solidity ^0.7.4;
.\interfaces\IPoolStake.sol:pragma solidity ^0.7.4;
.\interfaces\IPoolStrategy.sol:pragma solidity ^0.7.4;
.\interfaces\IRemove.sol:pragma solidity ^0.7.4;
.\interfaces\ISherlock.sol:pragma solidity ^0.7.4;
.\interfaces\ISherX.sol:pragma solidity ^0.7.4;
.\interfaces\ISherXERC20.sol:pragma solidity ^0.7.1;
.\interfaces\IStrategy.sol:pragma solidity ^0.7.4;
.\interfaces\aaveV2\DataTypes.sol:pragma solidity ^0.7.4;
.\interfaces\aaveV2\IAaveDistributionManager.sol:pragma solidity 0.7.6;
.\interfaces\aaveV2\IAaveGovernanceV2.sol:pragma solidity ^0.7.4;
.\interfaces\aaveV2\IAaveIncentivesController.sol:pragma solidity 0.7.6;
.\interfaces\aaveV2\IAToken.sol:pragma solidity ^0.7.4;
.\interfaces\aaveV2\IExecutorWithTimelock.sol:pragma solidity ^0.7.4;
.\interfaces\aaveV2\IGovernanceV2Helper.sol:pragma solidity ^0.7.4;
.\interfaces\aaveV2\ILendingPool.sol:pragma solidity ^0.7.4;
.\interfaces\aaveV2\ILendingPoolAddressesProvider.sol:pragma solidity ^0.7.4;
.\interfaces\aaveV2\IProposalValidator.sol:pragma solidity ^0.7.4;
.\interfaces\aaveV2\IStakeAave.sol:pragma solidity ^0.7.4;
.\interfaces\aaveV2\MockAave.sol:pragma solidity ^0.7.4;
.\libraries\LibPool.sol:pragma solidity ^0.7.4;
.\libraries\LibSherX.sol:pragma solidity ^0.7.4;
.\libraries\LibSherXERC20.sol:pragma solidity ^0.7.4;
.\storage\GovStorage.sol:pragma solidity ^0.7.0;
.\storage\PayoutStorage.sol:pragma solidity ^0.7.0;
.\storage\PoolStorage.sol:pragma solidity ^0.7.0;
.\storage\SherXERC20Storage.sol:pragma solidity ^0.7.1;
.\storage\SherXStorage.sol:pragma solidity ^0.7.0;
.\strategies\AaveV2.sol:pragma solidity ^0.7.4;
.\util\ERC20Mock.sol:pragma solidity ^0.7.4;
.\util\Import.sol:pragma solidity ^0.7.4;
.\util\RemoveMock.sol:pragma solidity ^0.7.4;
.\util\StrategyMock.sol:pragma solidity ^0.7.4;

## Tools Used
grep

## Recommended Mitigation Steps

Use the same solidity version everywhere

