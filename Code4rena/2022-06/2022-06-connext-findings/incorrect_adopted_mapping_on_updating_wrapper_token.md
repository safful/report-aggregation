## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Incorrect Adopted mapping on updating wrapper token](https://github.com/code-423n4/2022-06-connext-findings/issues/86) 

# Lines of code

https://github.com/code-423n4/2022-06-connext/blob/main/contracts/contracts/core/connext/facets/AssetFacet.sol#L100


# Vulnerability details

## Issue

1. Admin can call setWrapper function to setup a new wrapper Y instead of old wrapper X

2. This becomes a problem for any old asset which was setup during setupAsset call where s.canonicalToAdopted[_canonical.id]  will still point to old wrapper X instead of Y

## Recommendation
If wrapper is changed then all variables storing this wrapper should also update

