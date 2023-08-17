## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- edited-by-warden

# [Proposals can be bricked and Auctions stalled by bad settings](https://github.com/code-423n4/2022-09-nouns-builder-findings/issues/482) 

# Lines of code

https://github.com/code-423n4/2022-09-nouns-builder/blob/main/src/governance/governor/Governor.sol#L588


# Vulnerability details

## Impact

The protocol assumes founders and proposals will set sane settings. However there are some settings that if set incorrectly will block proposals from being created or succeeding and block auctions from completing.

This vulnerability has a low likelihood of occurrence as the outcome is not in the interest of the community. However the possibility exists if there is some misunderstanding or miscalculation. If a bad setting is allowed the impact is high.

## Proof of Concept

### Bricking governance proposals

**Governor settings.quorumThresholdBps > 10_000**

If `quorumThresholdBps` is set above 10_000 then it would be impossible to get enough votes to succeed.

Without being able to execute a proposal the setting itself could never be fixed.

**Governor settings.proposalThresholdBps > 10_000**

If `proposalThresholdBps` is set above 10_000 then it would be impossible to submit a proposal.

Without being able to submit a proposal the setting itself could never be fixed.

### Stalling a governance proposal

**Treasury settings.delay**

A very large value for `delay` would prevent a proposal from being executed.

For example 1000 years easily fits into `delay` and would result in a 1000 year wait before being able to execute.

A governance proposal could fix this property for future proposals but any proposal created with the large `delay` would remain stuck.

### Stalling the auction

**Auction settings.duration**

The `duration` value is in seconds and any value up to type(uint40).max is permitted.

That is `1099511627775` seconds which is > 48000 years.

A large value like this would stop the auction from ever ending and thus stop new NFTs from being minted.

A governance proposal could fix this setting but ideally a very large `duration` would be blocked.

**Auction settings.timeBuffer**

Similar to duration but applies to the auction endTime extention.

So the auction could be extended a number of years for example.

## Tools Used

Manual review.

## Recommended Mitigation Steps

Implement reasonable range bounds reverting where appropriate. In particular for the above apply:
- Governor settings `quorumThresholdBps` <= 10_000
- Governor settings `proposalThresholdBps` <= 10_000
- Treasury settings `delay` <= 6 months
- Auction settings `duration` <= 6 months
- Auction settings `timeBuffer` <= 6 months

Add these checks to the `initialize()` functions and in the setter / update functions where these individual settings properties can be updated.
