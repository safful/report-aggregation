## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed
- resolved

# [costly-loop](https://github.com/code-423n4/2021-06-realitycards-findings/issues/17) 

# Handle

heiho1


# Vulnerability details

## Impact

RCMarket#initialize(uint256,uint32[],uint256,uint256,address,address,address[],address,string) has a potentially expensive loop that modifies state continually over an indeterminate number of cards.

## Proof of Concept

https://github.com/code-423n4/2021-06-realitycards/blob/86a816abb058cc0ed9b6f5c4a8ad146f22b8034c/contracts/RCMarket.sol#L252

## Tools Used

Slither

## Recommended Mitigation Steps

Potentially a gas-expensive loop because of arbitrary length of _cardAffiliateAddresses possibly assigning to state variable cardAffiliateCut multiple times.
* It appears that the loop may be exited on the first cardAffiliateCut = 0 to optimize gas
* Alternatively a local variable may be assigned temporarily and then assigned to state: https://github.com/crytic/slither/wiki/Detector-Documentation#costly-operations-inside-a-loop


