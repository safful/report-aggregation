## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Only the `state()` of the latest proposal can be checked](https://github.com/code-423n4/2022-06-canto-findings/issues/254) 

# Lines of code

https://github.com/Plex-Engineer/lending-market/blob/755424c1f9ab3f9f0408443e6606f94e4f08a990/contracts/Governance/GovernorBravoDelegate.sol#L115


# Vulnerability details

## Impact

`state()` function cannot view the state from any proposal except for the latest one.

## Proof of Concept

```solidity
require(proposalCount >= proposalId && proposalId > initialProposalId, "GovernorBravo::state: invalid proposal id");
```

Currently `proposalCount` needs to be bigger or equal to `proposalId`. 
Assuming `proposalId` is incremented linearly in conjunction with `proposalCount`, this implies only the most recent `proposalId` will pass the `require()` check above. All other proposals will not be able to have their states checked via this function. 

## Tools Used
Manual Review.

## Recommended Mitigation Steps

Change above function to `proposalCount <= proposalId` (assuming `proposalId` is set linearly, which currently is not enforced by code).


