## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [uint256 is always >= 0](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/133) 

# Handle

0x0x0x


# Vulnerability details

## Impact
Gas optimization

## Proof of Concept

On Swap.sol at L190 and L191, it is checked that whether _a and _a2 is bigger equal to 0. Since they are both uint256, this condition is always satisfied. Therefore, those conditions are not required.

## Tools Used

Manual analysis

