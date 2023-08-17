## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed

# [Attacker can DOS private party by donating ETH then calling buy](https://github.com/code-423n4/2022-09-party-findings/issues/196) 

# Lines of code

https://github.com/PartyDAO/party-contracts-c4/blob/3896577b8f0fa16cba129dc2867aba786b730c1b/contracts/crowdfund/BuyCrowdfund.sol#L98-L116


# Vulnerability details

## Impact

Party is DOS'd and may potentially lose access to NFT

## Proof of Concept

[Crowdfund.sol#L280-L298](https://github.com/PartyDAO/party-contracts-c4/blob/3896577b8f0fa16cba129dc2867aba786b730c1b/contracts/crowdfund/Crowdfund.sol#L280-L298)

    party = party_ = partyFactory
        .createParty(
            address(this),
            Party.PartyOptions({
                name: name,
                symbol: symbol,
                governance: PartyGovernance.GovernanceOpts({
                    hosts: governanceOpts.hosts,
                    voteDuration: governanceOpts.voteDuration,
                    executionDelay: governanceOpts.executionDelay,
                    passThresholdBps: governanceOpts.passThresholdBps,
                    totalVotingPower: _getFinalPrice().safeCastUint256ToUint96(),
                    feeBps: governanceOpts.feeBps,
                    feeRecipient: governanceOpts.feeRecipient
                })
            }),
            preciousTokens,
            preciousTokenIds
        );

[BuyCrowdfundBase.sol#L166-L173](https://github.com/PartyDAO/party-contracts-c4/blob/3896577b8f0fa16cba129dc2867aba786b730c1b/contracts/crowdfund/BuyCrowdfundBase.sol#L166-L173)

    function _getFinalPrice()
        internal
        override
        view
        returns (uint256)
    {
        return settledPrice;
    }

When BuyCrowdFund.sol successfully completes a buy, totalVotingPower is set to _getFinalPrice which in the case of BuyCrowdFundBase.sol returns the price at which the NFT was purchased. totalVotingPower is used by the governance contract to determine the number of votes needed for a proposal to pass. If there are not enough claimable votes to meet that threshold then the party is softlocked because it can't pass any proposals. An attacker could exploit this to DOS even a private party with the following steps:

1. Wait for party to be filled to just under quorum threshold
2. Donate ETH to the crowdfund contract
3. Call BuyCrowdFund.sol#buy. Since it is unpermissioned even for parties with a gatekeeper, the call won't revert

Since the voting power for the final amount of ETH cannot be claimed, the party is now softlocked. If emergencyExecuteDisabled is true then the party will be permanantly locked and the NFT would effectively be burned. If emergencyExecuteDisabled is false then users would have to wait for PartyDAO to reclaim the NFT.

## Tools Used

## Recommended Mitigation Steps

Permission to call BuyCrowdFund.sol#buy should be gated if there is a gatekeeper