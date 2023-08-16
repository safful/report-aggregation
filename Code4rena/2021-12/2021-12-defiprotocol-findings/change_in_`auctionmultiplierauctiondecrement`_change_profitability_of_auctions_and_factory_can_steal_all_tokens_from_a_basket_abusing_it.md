## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Change in `auctionMultiplier/auctionDecrement` change profitability of auctions and factory can steal all tokens from a basket abusing it](https://github.com/code-423n4/2021-12-defiprotocol-findings/issues/145) 

# Handle

0x0x0x


# Vulnerability details

When factory changes `auctionMultiplier` or `auctionDecrement` profitability of bonded auctions change. There is no protection against this behaviour. Furthermore, factory owners can decide to get all tokens from baskets where they are bonded for the auction.

## Proof of concept

1- Factory owners call `bondForRebalance` for an auction.

2- Factory owners sets `auctionMultiplier` as 0 and `auctionDecrement` as maximum value

3- `settleAuction` is called. `newRatio = 0`, since `a = b = 0`. All tokens can be withdrawn with this call, since `tokensNeeded = 0`.

## Extra notes

Furthermore, even the factory owners does not try to scam users. In case `auctionMultiplier` or `auctionDecrement` is changed, all current `auctionBonder` from `Auctions` can only call `settleAuction` with different constraints. Because of different constraints, users/bonder will lose/gain funds.

## Mitigation step

Save `auctionDecrement` and `auctionMultiplier` to global variables in `Auction.sol`, when `startAuction` is called.

