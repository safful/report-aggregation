## Tags

- bug
- sponsor confirmed
- 1 (Low Risk)

# [Contradiction in comment/require statement](https://github.com/code-423n4/2021-10-tracer-findings/issues/7) 

# Handle

loop


# Vulnerability details

The comment for the `withdrawQuote()` function states 'Pool must not be paused'. Require statement requires paused to be true.

## Impact
Comment seems to be wrong, so no direct impact on functioning of protocol.

## Proof of Concept
Comment:
https://github.com/tracer-protocol/perpetual-pools-contracts/blob/646360b0549962352fe0c3f5b214ff8b5f73ba51/contracts/implementation/LeveragedPool.sol#L359

Require:
https://github.com/tracer-protocol/perpetual-pools-contracts/blob/646360b0549962352fe0c3f5b214ff8b5f73ba51/contracts/implementation/LeveragedPool.sol#L363

