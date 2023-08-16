## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Pool.sol: swapTo() should not be payable](https://github.com/code-423n4/2021-07-spartan-findings/issues/46) 

# Handle

hickuphh3


# Vulnerability details

### Impact

The `swapTo()` function should not be payable since the WBNB-SPARTA pool should not receive BNB, but WBNB. The router swap functions handles the wrapping and unwrapping of BNB.

Furthermore, the `swapTo()` will not detect any deposited BNB, so any swapTo() calls that have msg.value > 0 will have their BNB permanently locked in the pool contract. 

### Recommended Mitigation Steps

Remove `payable` keyword in `swapTo()`.

