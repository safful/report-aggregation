## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [`mintTo` arguments order](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/257) 

# Handle

0xsanson


# Vulnerability details

## Impact
In Basket.sol, there is a function `mintTo(uint256 amount, address to)`.
It's best practice to use as first argument `to`, and as second `amount`; see also the order used in L84 (_mint(to, amount)) and L86 (Minted(to, amount)).

## Proof of Concept
https://github.com/code-423n4/2021-09-defiProtocol/blob/main/contracts/contracts/Basket.sol#L76

## Tools Used
editor

## Recommended Mitigation Steps
Consider switching the arguments (also don't forget to change the calls to the function).

