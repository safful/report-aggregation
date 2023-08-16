## Tags

- bug
- QA (Quality Assurance)
- sponsor confirmed

# [QA Report](https://github.com/code-423n4/2022-02-redacted-cartel-findings/issues/123) 


1. Floating pragma is used in all the contracts

As different compiler versions have critical behavior specifics if the contracts get accidentally deployed using another compiler version compared to the one they were tested with, various types of undesired behavior can be introduced

## Proof of Concept

`pragma solidity ^0.8.0` is used in all the system contracts, for example:

https://github.com/code-423n4/2022-02-redacted-cartel/blob/main/contracts/BribeVault.sol#L2

https://github.com/code-423n4/2022-02-redacted-cartel/blob/main/contracts/RewardDistributor.sol#L2

## Recommended Mitigation Steps

Consider fixing the version to 0.8.x across all the codebase, for example set x to 10


2. Most of events in BribeVault, TokemakBribe and ThecosomataETH aren't indexed

## Impact

Filtering on not indexed events is disabled, which makes it harder to programmatically use and analyze the system

## Proof of Concept

The following events are not indexed:

https://github.com/code-423n4/2022-02-redacted-cartel/blob/main/contracts/BribeVault.sol#L37-57

https://github.com/code-423n4/2022-02-redacted-cartel/blob/main/contracts/TokemakBribe.sol#L42-43

https://github.com/code-423n4/2022-02-redacted-cartel/blob/main/contracts/ThecosomataETH.sol#L44-48

## Recommended Mitigation Steps

Consider making all events ids and addresses indexed to improve their usability


3. bytes32.length check is redundant and doesn't rule zero values out

## Impact

bytes32.length do not control for bogus values as the bytes32 length is fixed to 32. This way, for example, depositBribe can be successfully called with any identifiers, for example `0x0000000000000000000000000000000000000000000000000000000000000000` id will not be reverted

## Proof of Concept

bytes32 is a fixed size array, which length will always be 32, but it is checked for positive length anyway in BribeVault deposit functions:

https://github.com/code-423n4/2022-02-redacted-cartel/blob/main/contracts/BribeVault.sol#L171-172

https://github.com/code-423n4/2022-02-redacted-cartel/blob/main/contracts/BribeVault.sol#L218-219

## Recommended Mitigation Steps

If non-zero check is desired, `!= 0` can suffice (will revert on `0x0000000000000000000000000000000000000000000000000000000000000000`):

Now:
```
require(bribeIdentifier.length > 0, "Invalid bribeIdentifier");
```

To be:
```
require(bribeIdentifier != 0, "Invalid bribeIdentifier");
```
