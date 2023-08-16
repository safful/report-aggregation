## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- resolved

# [non existing function returns](https://github.com/code-423n4/2021-05-fairside-findings/issues/10) 

# Handle

gpersoon


# Vulnerability details

## Impact
The functions castVote and  castVoteBySig of FairSideDAO.sol have no "returns" parameters,
however they do call "return" at the end of the function.

This is confusing for the readers of the code.

## Proof of Concept
// https://github.com/code-423n4/2021-05-fairside/blob/main/contracts/dao/FairSideDAO.sol#L443
 function castVote(uint256 proposalId, bool support) public {
        return _castVote(msg.sender, proposalId, support);
    }

 function castVoteBySig( .. ) public {
       ...
       return _castVote(signatory, proposalId, support);
    }

## Tools Used
Editor

## Recommended Mitigation Steps

Remove the "return" statements from castVote and castVoteBySig

