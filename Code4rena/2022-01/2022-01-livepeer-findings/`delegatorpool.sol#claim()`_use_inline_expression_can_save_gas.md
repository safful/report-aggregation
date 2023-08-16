## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [`DelegatorPool.sol#claim()` Use inline expression can save gas](https://github.com/code-423n4/2022-01-livepeer-findings/issues/131) 

# Handle

WatchPug


# Vulnerability details

https://github.com/livepeer/arbitrum-lpt-bridge/blob/ebf68d11879c2798c5ec0735411b08d0bea4f287/contracts/L2/pool/DelegatorPool.sol#L70-L78

```solidity
        // Calculate stake owed to delegator
        uint256 currTotalStake = pendingStake();
        uint256 owedStake = (currTotalStake * _stake) /
            (initialStake - claimedInitialStake);

        // Calculate fees owed to delegator
        uint256 currTotalFees = pendingFees();
        uint256 owedFees = (currTotalFees * _stake) /
            (initialStake - claimedInitialStake);
```

The local variable `currTotalStake`, `currTotalFees` is used only once. Making the expression inline can save gas.

Similar issue exists in `L2Migrator.sol#claimStake()`, `L1Migrator.sol#migrateETH()`, `L1Migrator.sol#migrateLPT()`, `L1ArbitrumMessenger.sol#onlyL2Counterpart()`.

### Recommendation

Change to:

```solidity
        // Calculate stake owed to delegator
        uint256 owedStake = (pendingStake() * _stake) /
            (initialStake - claimedInitialStake);

        // Calculate fees owed to delegator
        uint256 owedFees = (pendingFees() * _stake) /
            (initialStake - claimedInitialStake);
```


