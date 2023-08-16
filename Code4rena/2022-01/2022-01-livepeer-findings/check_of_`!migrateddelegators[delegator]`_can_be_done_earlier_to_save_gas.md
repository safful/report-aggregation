## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Check of `!migratedDelegators[delegator]` can be done earlier to save gas](https://github.com/code-423n4/2022-01-livepeer-findings/issues/127) 

# Handle

WatchPug


# Vulnerability details

When there are multiple checks, adjusting the sequence to allow the tx to fail earlier can save some gas.

Checks using less gas should be executed earlier than those with higher gas costs, to avoid unnecessary storage read, arithmetic operations, etc when it reverts.

For example:

https://github.com/livepeer/arbitrum-lpt-bridge/blob/ebf68d11879c2798c5ec0735411b08d0bea4f287/contracts/L2/gateway/L2Migrator.sol#L255-L275

```solidity
        require(
            claimStakeEnabled,
            "L2Migrator#claimStake: CLAIM_STAKE_DISABLED"
        );

        IMerkleSnapshot merkleSnapshot = IMerkleSnapshot(merkleSnapshotAddr);

        address delegator = msg.sender;
        bytes32 leaf = keccak256(
            abi.encodePacked(delegator, _delegate, _stake, _fees)
        );

        require(
            merkleSnapshot.verify(keccak256("LIP-73"), _proof, leaf),
            "L2Migrator#claimStake: INVALID_PROOF"
        );

        require(
            !migratedDelegators[delegator],
            "L2Migrator#claimStake: ALREADY_MIGRATED"
        );
```

The check of `!migratedDelegators[delegator]` can be done earlier to avoid reading from storage when `migratedDelegators[delegator] == true`.

## Recommendation

Change to:

```solidity
        require(
            claimStakeEnabled,
            "L2Migrator#claimStake: CLAIM_STAKE_DISABLED"
        );

        address delegator = msg.sender;
        require(
            !migratedDelegators[delegator],
            "L2Migrator#claimStake: ALREADY_MIGRATED"
        );

        IMerkleSnapshot merkleSnapshot = IMerkleSnapshot(merkleSnapshotAddr);
        require(
            merkleSnapshot.verify(keccak256("LIP-73"), _proof, leaf),
            "L2Migrator#claimStake: INVALID_PROOF"
        );

        bytes32 leaf = keccak256(
            abi.encodePacked(delegator, _delegate, _stake, _fees)
        );
```

