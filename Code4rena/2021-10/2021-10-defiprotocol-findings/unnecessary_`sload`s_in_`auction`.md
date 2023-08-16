## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Unnecessary `SLOAD`s in `Auction`](https://github.com/code-423n4/2021-10-defiprotocol-findings/issues/33) 

# Handle

pants


# Vulnerability details

The functions `Auction.bondForRebalance()`, `Auction.settleAuction()`, `Auction.bondBurn()` and `Auction.withdrawBounty()` read values from storage multiple times instead of caching them in local variables:
- `Auction.bondForRebalance()` reads `bondAmount` twice.
- `Auction.settleAuction()` reads `bondBlock` twice, `basket` 8 times and `factory` twice.
- `Auction.bondBurn()` reads `basket` twice and `bondAmount` twice.
- `Auction.withdrawBounty()` reads `bounty.token` twice and `bounty.amount` twice.

## Impact
Storage reads are much more expensive than reading local variables.

## Tool Used
Manual code review.

## Recommended Mitigation Steps
Read these values from storage once, cache them in local variables and then read them again from the local variables.

