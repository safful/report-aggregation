## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [++i is more gas efficient than i++ in loops forwarding](https://github.com/code-423n4/2021-12-amun-findings/issues/108) 

# Handle

defsec


# Vulnerability details

## Impact

++i is more gas efficient than i++ in loops forwarding.

## Proof of Concept

1. Navigate to the following contracts.

"https://github.com/code-423n4/2021-12-amun/blob/main/contracts/basket/contracts/callManagers/RebalanceManager.sol#L218"

"https://github.com/code-423n4/2021-12-amun/blob/main/contracts/basket/contracts/callManagers/RebalanceManager.sol#L234"

"https://github.com/code-423n4/2021-12-amun/blob/main/contracts/basket/contracts/callManagers/RebalanceManagerV2.sol#L155"

"https://github.com/code-423n4/2021-12-amun/blob/main/contracts/basket/contracts/callManagers/RebalanceManagerV3.sol#L166"

"https://github.com/code-423n4/2021-12-amun/blob/main/contracts/basket/contracts/factories/PieFactoryContract.sol#L88"

"https://github.com/code-423n4/2021-12-amun/blob/main/contracts/basket/contracts/facets/Call/CallFacet.sol#L55"

"https://github.com/code-423n4/2021-12-amun/blob/main/contracts/basket/contracts/facets/Basket/BasketFacet.sol#L50"

"https://github.com/code-423n4/2021-12-amun/blob/main/contracts/basket/contracts/facets/Basket/BasketFacet.sol#L160"

"https://github.com/code-423n4/2021-12-amun/blob/main/contracts/basket/contracts/facets/Basket/BasketFacet.sol#L321"

"https://github.com/code-423n4/2021-12-amun/blob/main/contracts/basket/contracts/facets/Basket/BasketFacet.sol#L348"

"https://github.com/code-423n4/2021-12-amun/blob/main/contracts/basket/contracts/facets/Basket/BasketFacet.sol#L381"

"https://github.com/code-423n4/2021-12-amun/blob/main/contracts/basket/contracts/singleJoinExit/SingleNativeTokenExit.sol#L69"


## Tools Used

Code Review

## Recommended Mitigation Steps

It is  recommend to use unchecked{++i} and change i declaration to uint256.

