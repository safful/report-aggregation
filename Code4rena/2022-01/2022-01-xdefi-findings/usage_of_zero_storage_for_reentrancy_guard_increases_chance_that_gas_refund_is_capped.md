## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Usage of zero storage for reentrancy guard increases chance that gas refund is capped](https://github.com/code-423n4/2022-01-xdefi-findings/issues/1) 

# Handle

TomFrenchBlockchain


# Vulnerability details

## Impact

Reduction of potential gas refunds.

## Proof of Concept

The reentrancy guard variable is initially set to zero, set to a nonzero value and then reset to zero:

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/4a9cc8b4918ef3736229a5cc5a310bdc17bf759f/contracts/security/ReentrancyGuard.sol#L29-L35

We then have to the higher cost for writing to clean storage rather than dirty storage (which is then refunded). This is not recommended as it can cause the size of the gas refunded to users to be capped. For more info see the OZ implementation:

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/4a9cc8b4918ef3736229a5cc5a310bdc17bf759f/contracts/security/ReentrancyGuard.sol#L29-L35

## Recommended Mitigation Steps

Change from 0->1->0 to 1->2->1

