## Tags

- bug
- 0 (Non-critical)
- resolved
- sponsor confirmed

# [Missing event for admin function setBaseURI](https://github.com/code-423n4/2022-01-xdefi-findings/issues/16) 

# Handle

BouSalman


# Vulnerability details

## Vulnerability description
In Contract **XDEFIDistribution** the function **setBaseURI** is missing an event for this admin functionality.

## Impact
Users can't monitor admin changes done to the contract to reflect it in their clients.

## Proof of Concept
https://github.com/XDeFi-tech/xdefi-distribution/blob/3856a42df295183b40c6eee89307308f196612fe/contracts/XDEFIDistribution.sol#L73

## Tools Used
manual code review.

## Recommended Mitigation Steps
create event for base URI changes and emit it.

