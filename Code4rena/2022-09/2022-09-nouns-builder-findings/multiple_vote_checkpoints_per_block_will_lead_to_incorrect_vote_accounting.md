## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Multiple vote checkpoints per block will lead to incorrect vote accounting](https://github.com/code-423n4/2022-09-nouns-builder-findings/issues/185) 

# Lines of code

https://github.com/code-423n4/2022-09-nouns-builder/blob/7e9fddbbacdd7d7812e912a369cfd862ee67dc03/src/lib/token/ERC721Votes.sol#L252-L253


# Vulnerability details

Voting power for each NFT owner is persisted within timestamp-dependent checkpoints. Every voting power increase or decrease is recorded. However, the implementation of `ERC721Votes` creates separate checkpoints with the same timestamp for each interaction, even when the interactions happen in the same block/timestamp.

## Impact

Checkpoints with the same `timestamp` will cause issues within the `ERC721Votes.getPastVotes(..)` function and will return incorrect votes for a given `_timestamp`.

## Proof of Concept

[lib/token/ERC721Votes.sol#L252-L253](https://github.com/code-423n4/2022-09-nouns-builder/blob/7e9fddbbacdd7d7812e912a369cfd862ee67dc03/src/lib/token/ERC721Votes.sol#L252-L253)

```solidity
/// @dev Records a checkpoint
/// @param _account The account address
/// @param _id The checkpoint id
/// @param _prevTotalVotes The account's previous voting weight
/// @param _newTotalVotes The account's new voting weight
function _writeCheckpoint(
    address _account,
    uint256 _id,
    uint256 _prevTotalVotes,
    uint256 _newTotalVotes
) private {
    // Get the pointer to store the checkpoint
    Checkpoint storage checkpoint = checkpoints[_account][_id];

    // Record the updated voting weight and current time
    checkpoint.votes = uint192(_newTotalVotes);
    checkpoint.timestamp = uint64(block.timestamp);

    emit DelegateVotesChanged(_account, _prevTotalVotes, _newTotalVotes);
}
```

**Consider the following example and the votes checkpoint snapshots:**

_Note: Bob owns a smart contract used to interact with the protocol_

**Transaction 0:** Bob's smart contract receives 1 NFT through minting (1 NFT equals 1 vote)

| Checkpoint Index | Timestamp | Votes |
| ---------------- | --------- | ----- |
| 0                | 0         | 1     |

**Transaction 1:** Bob's smart contract receives one more NFT through minting

| Checkpoint Index | Timestamp | Votes |
| ---------------- | --------- | ----- |
| 0                | 0         | 1     |
| 1                | 1         | 2     |

**Transaction 1:** Within the same transaction 1, Bob's smart-contract delegates 2 votes to Alice

| Checkpoint Index | Timestamp | Votes |
| ---------------- | --------- | ----- |
| 0                | 0         | 1     |
| 1                | 1         | 2     |
| 2                | 1         | 0     |

**Transaction 1:** Again within the same transaction 1, Bob's smart contract decides to reverse the delegation and self-delegates

| Checkpoint Index | Timestamp | Votes |
| ---------------- | --------- | ----- |
| 0                | 0         | 1     |
| 1                | 1         | 2     |
| 2                | 1         | 0     |
| 3                | 1         | 2     |

**Transaction 1:** Bob's smart contract buys one more NFT

| Checkpoint Index | Timestamp | Votes |
| ---------------- | --------- | ----- |
| 0                | 0         | 1     |
| 1                | 1         | 2     |
| 2                | 1         | 0     |
| 3                | 1         | 2     |
| 4                | 2         | 3     |

Bob now wants to vote (via his smart contract) on a governance proposal that has been created on `timeCreated = 1` (timestamp 1).

Internally, the `Governor._castVote` function determines the voter's weight by calling `getVotes(_voter, proposal.timeCreated)`.

[governance/governor/Governor.sol#L275](https://github.com/code-423n4/2022-09-nouns-builder/blob/7e9fddbbacdd7d7812e912a369cfd862ee67dc03/src/governance/governor/Governor.sol#L275)

```solidity
weight = getVotes(_voter, proposal.timeCreated);
```

`getVotes` calls `ERC721.getPastVotes` internally:

[governance/governor/Governor.sol#L462](https://github.com/code-423n4/2022-09-nouns-builder/blob/7e9fddbbacdd7d7812e912a369cfd862ee67dc03/src/governance/governor/Governor.sol#L462)

```solidity
function getVotes(address _account, uint256 _timestamp) public view returns (uint256) {
    return settings.token.getPastVotes(_account, _timestamp);
}
```

`ERC721.getPastVotes(..., 1)` tries to find the checkpoint within the `while` loop:

| # Iteration | `low` | `middle` | `high` |
| ----------- | ----- | -------- | ------ |
| 0           | 0     | 2        | 4      |

The `middle` checkpoint with index `2` matches the given timestamp `1` and returns `0` votes. This is incorrect, as Bob has 2 votes. Bob is not able to vote properly.

_(Please be aware that this is just one of many examples of how this issue can lead to incorrect vote accounting. In other cases, NFT owners could have more voting power than they are entitled to)_

## Tools Used

Manual review

## Recommended mitigation steps

Consider batching multiple checkpoints writes per block/timestamp similar to how NounsDAO records checkpoints.
