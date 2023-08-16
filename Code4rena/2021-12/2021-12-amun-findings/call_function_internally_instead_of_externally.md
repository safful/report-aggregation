## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Call function internally instead of externally](https://github.com/code-423n4/2021-12-amun-findings/issues/270) 

# Handle

Czar102


# Vulnerability details

## Impact
To save gas it is recommended it call function internally, if possible.

## Proof of Concept
Function [`BasketFacet::getLock()`](https://github.com/code-423n4/2021-12-amun/blob/main/contracts/basket/contracts/facets/Basket/BasketFacet.sol#L282-L286) is defined externally and calls from [`BasketFacet::joinPool(...)`](https://github.com/code-423n4/2021-12-amun/blob/main/contracts/basket/contracts/facets/Basket/BasketFacet.sol#L148) and [`BasketFacet::exitPool(...)`](https://github.com/code-423n4/2021-12-amun/blob/main/contracts/basket/contracts/facets/Basket/BasketFacet.sol#L194) are not internal, but message calls.

The same applies to function [`BasketFacet::getCap()`](https://github.com/code-423n4/2021-12-amun/blob/main/contracts/basket/contracts/facets/Basket/BasketFacet.sol#L328-L330) usage in [`BasketFacet::joinPool(...)`](https://github.com/code-423n4/2021-12-amun/blob/main/contracts/basket/contracts/facets/Basket/BasketFacet.sol#L154).

## Recommended Mitigation Steps
Consider changing visibility of `BasketFacet::getLock()` to public and calling the above function internally. Alternative solution shall be implemented with `BasketFacet::getCap()`.


