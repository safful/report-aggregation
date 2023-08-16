## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [hard to clear balance](https://github.com/code-423n4/2021-10-badgerdao-findings/issues/24) 

# Handle

jonah1005


# Vulnerability details

## Impact
The contract does not allow users to transfer by share. It's hard for users to clear out all the shares.
There will be users using this token with Metamask. There's likely the `pricePerShare` would increase after the user sends transactions.
I consider this is a medium-risk issue.

## Proof of Concept
[WrappedIbbtc.sol#L110-L118](https://github.com/code-423n4/2021-10-badgerdao/blob/main/contracts/WrappedIbbtc.sol#L110-L118)

## Tools Used

## Recommended Mitigation Steps
I consider a new `transferShares` beside the original `transfer()` would build a better UX.
I consider sushi's bento box would be a good ref [BentoBox.sol](https://github.com/sushiswap/bentobox/blob/master/contracts/BentoBox.sol)

