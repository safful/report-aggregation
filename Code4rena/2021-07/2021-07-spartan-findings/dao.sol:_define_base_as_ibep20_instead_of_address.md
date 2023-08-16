## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Dao.sol: Define BASE as iBEP20 instead of address](https://github.com/code-423n4/2021-07-spartan-findings/issues/60) 

# Handle

hickuphh3


# Vulnerability details

### Impact

`BASE` is defined as an `address` type, but is casted as `iBEP20` in almost every instance within the Dao contract, and in numerous instances in many other contracts as well. It would therefore be better to define it as `iBEP20` instead, to avoid casting.

### Recommended Mitigation Steps

Change `address public BASE;` to `iBEP public BASE`. Castings of `BASE` to `iBEP20` can be removed subsequently.

