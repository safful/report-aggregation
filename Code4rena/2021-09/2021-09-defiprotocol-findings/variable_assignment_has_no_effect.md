## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Variable assignment has no effect](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/124) 

# Handle

nikitastupin


# Vulnerability details

Here https://github.com/code-423n4/2021-09-defiProtocol/blob/e6dcf43a2f03aa65e04f0edc8ed1d7272677fabe/contracts/contracts/Auction.sol#L143-L143 the `bounty` variable is copied from Storage to Memory. Later it's assigned to false https://github.com/code-423n4/2021-09-defiProtocol/blob/e6dcf43a2f03aa65e04f0edc8ed1d7272677fabe/contracts/contracts/Auction.sol#L147. However, this assignment has no effect because `bounty` variable located at Memory so it's basically just thrown away when loop iteration finishes.

I think the intention was to make the `bounty.active` false so the same bounty isn't claimed twice or more times https://github.com/code-423n4/2021-09-defiProtocol/blob/e6dcf43a2f03aa65e04f0edc8ed1d7272677fabe/contracts/contracts/Auction.sol#L144. However, the `bounty.active` will always be true because it never changes to false except for https://github.com/code-423n4/2021-09-defiProtocol/blob/e6dcf43a2f03aa65e04f0edc8ed1d7272677fabe/contracts/contracts/Auction.sol#L147 (which has no effect).

## Impact

I don't see the direct impact here, however it may arise with the future changes to the contracts.

## Proof of Concept

I'll write a PoC if needed.

## Recommended Mitigation Steps

Do `_bounties[bountyIds[i]].active = false` instead of `bounty.active = false` if you need this check or just remove `bounty.active = false` and `require(bounty.active)` lines to save a gas otherwise.

