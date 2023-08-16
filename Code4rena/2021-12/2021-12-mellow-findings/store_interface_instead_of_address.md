## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Store Interface instead of address](https://github.com/code-423n4/2021-12-mellow-findings/issues/21) 

# Handle

0x1f8b


# Vulnerability details

## Impact
Gas saving.

## Proof of Concept
Inside the contract `ChiefTrader` It's better to store the variable `protocolGovernance` as `IProtocolGovernance` because otherwise you need to cast it everytime.

## Tools Used
Manual review

## Recommended Mitigation Steps
Use `IProtocolGovernance` instead of address for store `protocolGovernance`

