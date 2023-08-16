## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [`public` functions in `Basket` can be `external`](https://github.com/code-423n4/2021-10-defiprotocol-findings/issues/13) 

# Handle

pants


# Vulnerability details

The `public` functions `Basket.mint()`, `Basket.burn()`, `Basket.changePublisher()`, `Basket.changeLicenseFee()`, `Basket.publishNewIndex()` and `Basket.deleteNewIndex()` are never called by `Basket`. Therefore, their visibility can be reduced to `external`.

## Impact
`external` functions are cheaper than `public` functions.

## Proof of Concept
https://gus-tavo-guim.medium.com/public-vs-external-functions-in-solidity-b46bcf0ba3ac

## Tool Used
Manual code review.

## Recommended Mitigation Steps
Define these functions as `public`.

