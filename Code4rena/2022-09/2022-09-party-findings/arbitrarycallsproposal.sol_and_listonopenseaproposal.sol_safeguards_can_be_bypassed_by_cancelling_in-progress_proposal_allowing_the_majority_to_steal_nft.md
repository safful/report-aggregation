## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed
- edited-by-warden

# [ArbitraryCallsProposal.sol and ListOnOpenseaProposal.sol safeguards can be bypassed by cancelling in-progress proposal allowing the majority to steal NFT](https://github.com/code-423n4/2022-09-party-findings/issues/153) 

# Lines of code

https://github.com/PartyDAO/party-contracts-c4/blob/3896577b8f0fa16cba129dc2867aba786b730c1b/contracts/proposals/ProposalExecutionEngine.sol#L183-L202


# Vulnerability details

Note: PartyDAO acknowledges that "canceling an InProgress proposal (mid-step) can leave the governance party in a vulnerable or undesirable state because there is no cleanup logic run during a cancel" in the "Known Issues / Topics" section of the contest readme. I still believe that this vulnerability needs to be mitigated as it can directly lead to loss of user funds.

## Impact

Majority vote can abuse cancel functionality to steal an NFT owned by the party

## Proof of Concept

ArbitraryCallsProposal.sol implements the following safeguards for arbitrary proposals that are not unanimous:

1. Prevents the ownership of any NFT changing during the call. It does this by checking the the ownership of all NFTs before and after the call.

2. Prevents calls that would change the approval status of any NFT. This is done by disallowing the "approve" and "setApprovalForAll" function selectors.

Additionally ListOnOpenseaProposal.sol implements the following safeguards:

1. NFTs are first listed for auction on Zora so that if they are listed for a very low price then the auction will keep them from being purchased at such a low price

2. At the end of the auction the approval is revoked when _cleanUpListing is called

These safeguards are ultimately ineffective though. The majority could use the following steps to steal the NFT:

1. Create ListOnOpenseaProposal with high sell price and short cancel delay

2. Vote to approve proposal with majority vote

3. Execute first step of proposal, listing NFT on Zora auction for high price

4. Wait for Zora auction to end since the auction price is so high that no one will buy it

5. Execute next step, listing the NFT on Opensea. During this step the contract grants approval of the NFT to the Opensea contract

6. Wait for cancelDelay to expire

7. Call PartyGovernance.sol#cancel. This will immediately terminate the Opensea bypassing _cleanUpListing and keeping the approval to the Opensea contract

8. Create ArbitraryCallsProposal.sol that lists the NFT on Opensea for virtually nothing. Since only approval selectors have been blacklisted and the NFT does not change ownership, the proposal does not need to be unanimous to execute.

9. Approve proposal and execute

10. Buy NFT

## Tools Used

Manual Review

## Recommended Mitigation Steps

When a proposal is canceled, it should call a proposal specific function that makes sure everything is cleaned up. NFTs delisted, approvals revoked, etc.