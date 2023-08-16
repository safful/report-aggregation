## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Auction.userClaimableArbTokens nonzero auction.finalPrice check is redundant](https://github.com/code-423n4/2021-11-malt-findings/issues/49) 

# Handle

hyh


# Vulnerability details

## Proof of Concept

As there is a check in the beginning of the function that includes the ```auction.finalPrice == 0``` condition:
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/Auction.sol#L285

## Recommended Mitigation Steps

The same condition down the line is never true and its check is redundant:
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/Auction.sol#L298

