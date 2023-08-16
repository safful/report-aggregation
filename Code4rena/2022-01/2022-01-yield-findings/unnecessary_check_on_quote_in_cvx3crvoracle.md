## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Unnecessary check on quote in Cvx3CrvOracle](https://github.com/code-423n4/2022-01-yield-findings/issues/79) 

# Handle

TomFrenchBlockchain


# Vulnerability details

## Impact
gas costs

## Proof of Concept

L116 of Cvx3CrvOracle enforces for the rest of the function call that `base == ethId <-> quote == cvx3CrvId`

https://github.com/code-423n4/2022-01-yield/blob/e946f40239b33812e54fafc700eb2298df1a2579/contracts/Cvx3CrvOracle.sol#L116

However on L137 we check both these conditions again.

https://github.com/code-423n4/2022-01-yield/blob/e946f40239b33812e54fafc700eb2298df1a2579/contracts/Cvx3CrvOracle.sol#L137

We could check just one of these and then rely on the require condition on 116 to enforce the other one. This will prevent us having to SLOAD `ethID` again

## Recommended Mitigation Steps

Change L137 to `if (base == cvx3CrvId) {`

