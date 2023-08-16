## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [`DelegatorPool.sol#claim()` Inaccurate check of `claimedInitialStake < initialStake`](https://github.com/code-423n4/2022-01-livepeer-findings/issues/190) 

# Handle

WatchPug


# Vulnerability details

In the current implementation of `DelegatorPool.sol#claim()`, it first requires `claimedInitialStake < initialStake`, or it throws an error of `DelegatorPool#claim: FULLY_CLAIMED`.

However, since it's an `onlyMigrator` function, the felicity of `_delegator` and `_stake` should be assured by the `Migrator` contract, otherwise, this `require` statement itself also can not prevent bad results caused by the wrong inputs.

Furthermore, even if the purpose of this `require` statement is to make sure that `claimedInitialStake` can never surpass the `initialStake`, the expression should be `claimedInitialStake + _stake <= initialStake` instead of `claimedInitialStake < initialStake`.

https://github.com/livepeer/arbitrum-lpt-bridge/blob/ebf68d11879c2798c5ec0735411b08d0bea4f287/contracts/L2/pool/DelegatorPool.sol#L58-L93

```solidity
function claim(address _delegator, uint256 _stake) external onlyMigrator {
    require(
        claimedInitialStake < initialStake,
        "DelegatorPool#claim: FULLY_CLAIMED"
    );

    // Calculate stake owed to delegator
    uint256 currTotalStake = pendingStake();
    uint256 owedStake = (currTotalStake * _stake) /
        (initialStake - claimedInitialStake);

    // Calculate fees owed to delegator
    uint256 currTotalFees = pendingFees();
    uint256 owedFees = (currTotalFees * _stake) /
        (initialStake - claimedInitialStake);

    // update claimed balance
    claimedInitialStake += _stake;

    // Transfer owed stake to the delegator
    transferBond(_delegator, owedStake);

    // Transfer owed fees to the delegator
    IBondingManager(bondingManager).withdrawFees(
        payable(_delegator),
        owedFees
    );

    emit Claimed(_delegator, owedStake, owedFees);
}
```

## Recommandation

Consider removing it or changing to:

```solidity
require(
    claimedInitialStake + _stake <= initialStake,
    "DelegatorPool#claim: FULLY_CLAIMED"
);
```

