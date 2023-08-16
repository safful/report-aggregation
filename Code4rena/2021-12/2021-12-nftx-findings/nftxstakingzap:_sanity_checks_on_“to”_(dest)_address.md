## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [NFTXStakingZap: Sanity checks on “to” (dest) address](https://github.com/code-423n4/2021-12-nftx-findings/issues/227) 

# Handle

GreyArt


# Vulnerability details

## Impact

The MarketplaceZap contract conducts a sanity check on the `to` address. One can therefore expect that this check is in place for the StakingZap contract as well.

We also suggest adding another check to ensure that the `to` address is not the StakingZap contract itself. Although there is a `rescue()` function to retrieve funds in these cases, it would be a hassle to do so. 

## Recommended Mitigation Steps

Include the sanity check(s) of the `to` address in the `addLiquidity*()` functions.

```jsx
require(to != address(0) && to != address(this));
```

