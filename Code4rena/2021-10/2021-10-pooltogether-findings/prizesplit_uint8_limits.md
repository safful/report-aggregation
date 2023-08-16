## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- resolved

# [PrizeSplit uint8 limits](https://github.com/code-423n4/2021-10-pooltogether-findings/issues/17) 

# Handle

gpersoon


# Vulnerability details

## Impact
In the contract PrizeSplit.sol, uint8 is used in a few places. This limits the addressable size of _prizeSplits.

In practice you would probably not split prizes in more than 256 ways, but checking for this is safer.

## Proof of Concept
https://github.com/pooltogether/v4-core/blob/master/contracts/prize-strategy/PrizeSplit.sol#L86-L87

function setPrizeSplit(PrizeSplitConfig memory _prizeSplit, uint8 _prizeSplitIndex)

https://github.com/pooltogether/v4-core/blob/master/contracts/prize-strategy/PrizeSplit.sol#L130-L140

for (uint8 index = 0; index < prizeSplitsLength; index++) {

## Tools Used

## Recommended Mitigation Steps
Add the following  to function setPrizeSplits:
   require(newPrizeSplitsLength <= type(uint8).max))

or replace uint8 with uint256 in PrizeSplit.sol


