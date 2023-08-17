## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Anyone can create Proposal Unigov Proposal-Store.sol](https://github.com/code-423n4/2022-06-canto-findings/issues/26) 

# Lines of code

https://github.com/Plex-Engineer/manifest/blob/688e9b4e7835854c22ef44b045d6d226b784b4b8/contracts/Proposal-Store.sol#L46
https://github.com/Plex-Engineer/lending-market/blob/b93e2867a64b420ce6ce317f01c7834a7b6b17ca/contracts/Governance/GovernorBravoDelegate.sol#L37


# Vulnerability details

## Impact
Proposal Store is used to store proposals that have already passed (https://code4rena.com/contests/2022-06-new-blockchain-contest#unigov-module-615-sloc) " Upon a proposal’s passing, the proposalHandler either deploys the ProposalStore contract (if it is not already deployed) or appends the proposal into the ProposalStore’s mapping ( uint ⇒ Proposal)"

But anyone can add proposals to the contract directly via AddProposal() function.

Unigov proposals can be queued and executed by anyone in GovernorBravoDelegate contract
https://github.com/Plex-Engineer/lending-market/blob/b93e2867a64b420ce6ce317f01c7834a7b6b17ca/contracts/Governance/GovernorBravoDelegate.sol#L37

## Proof of Concept
https://github.com/Plex-Engineer/manifest/blob/688e9b4e7835854c22ef44b045d6d226b784b4b8/contracts/Proposal-Store.sol#L46

## Recommended Mitigation Steps
Authorization checks for AddProposal, only governance module should be able to update

