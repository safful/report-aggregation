## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [`setupParticipant` function should be internal](https://github.com/code-423n4/2021-11-malt-findings/issues/183) 

# Handle

nathaniel


# Vulnerability details

## Impact
No vulnerability, however as `setupPartipant` would only ever be executed by the constructor in its deriving contracts, it would make sense if it was internal instead of public. If it was not executed in the constructor of the deriving contract, then at least it is safer with internal visibility.

## Proof of Concept
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/AuctionParticipant.sol#L30


