## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [No checks on target variable](https://github.com/code-423n4/2021-08-notional-findings/issues/29) 

# Handle

tensors


# Vulnerability details

## Impact
Lack of checks on target could lead to loss of funds.

## Proof of Concept
https://github.com/code-423n4/2021-08-notional/blob/4b51b0de2b448e4d36809781c097c7bc373312e9/contracts/external/governance/Reservoir.sol#L50

## Recommended Mitigation Steps
Require that target is non-zero.

