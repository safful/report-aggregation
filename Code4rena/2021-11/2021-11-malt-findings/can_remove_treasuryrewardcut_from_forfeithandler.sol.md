## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Can remove treasuryRewardCut from ForfeitHandler.sol](https://github.com/code-423n4/2021-11-malt-findings/issues/379) 

# Handle

harleythedog


# Vulnerability details

## Impact
In ForfeitHandler.sol, there are two values `swingTraderRewardCut ` and `treasuryRewardCut `, and these values always sum to 1000. Instead of having to go through all of the logic of setting these values independently and always ensuring that they sum to 1000, it would be simpler (and definitely save a lot of gas) if you simply removed everything related to `treasuryRewardCut` and always just used `1000-swingTraderRewardCut` in its place.

This also is more similar to what is done in StabilizerNode.sol where `treasuryCut` is simply what is left over after other components have taken their cut.

## Proof of Concept
See ForfeitHandler.sol here: https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/ForfeitHandler.sol

## Tools Used
Inspection

## Recommended Mitigation Steps
Simplify logic and save gas by removing `treasuryRewardCut`.

