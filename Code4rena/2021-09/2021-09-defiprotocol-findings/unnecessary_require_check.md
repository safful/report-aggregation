## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Unnecessary require check](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/273) 

# Handle

tensors


# Vulnerability details

## Impact
In L92 of Basket.sol there is an unnecessary require check that the user balance is greater than or equal to amount.
If the amount is larger than user balance then the _burn() method will fail, causing the function to revert anyway.

## Proof of Concept
https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Basket.sol#L92

## Recommended Mitigation Steps
Remove the unnecessary check.

