## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Dao.sol: newParamProposal takes in uint32 param](https://github.com/code-423n4/2021-07-spartan-findings/issues/61) 

# Handle

hickuphh3


# Vulnerability details

### Impact

`newParamProposal()` takes in a `uint32 param` as an input argument. The valid scenarios for this proposal are for changing the cooloff period and erasToEarn via the `changeCooloff()` and `changeEras()`. These functions however cast the `param` to `uint256` before assigning it to the relevant variable. 

We therefore have either of the following cases:

1.  `uint32 param` should be increased to `uint256 param`
2. `coolOffPeriod` and `erasToEarn` can be decreased in size to `uint32` instead of `uint256`. For further optimizations, these 2 variables should be grouped together so that they take up 1 storage slot instead of 2 separate ones.

