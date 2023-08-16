## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Re-entrancy in `Factory.createBasket`](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/219) 

# Handle

cmichel


# Vulnerability details

A basket creator can specify a custom token that allows them to re-enter in `Factory.createBasket`.

## Impact
As new auction and basket contracts are created every time, no cross-basket issues arise.
However, note that the official `BasketCreated` event is emitted for all of them, but only the last basket is stored for the `idNumber`.
This could lead to issues for some backend / frontend scripts that use the `BasketCreated` event.

## Recommended Mitigation Steps
Set `_proposals[idNumber].basket = address(newBasket);` immediately after the `newBasket` contract clone has been created to avoid the re-entrancy.


