## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [State variables in `Factory` can be `immutable`](https://github.com/code-423n4/2021-10-defiprotocol-findings/issues/11) 

# Handle

pants


# Vulnerability details

The state variables `Factory.auctionImpl` and `Factory.basketImpl` can be `immutable` since they are only set once, at the constructor.

## Impact
Reading from immutable state variables is much cheaper than from regular state variables.

## Proof of Concept
https://blog.soliditylang.org/2020/05/13/immutable-keyword/

## Tool Used
Manual code review.

## Recommended Mitigation Steps
Define these state variables as `immutable`.

