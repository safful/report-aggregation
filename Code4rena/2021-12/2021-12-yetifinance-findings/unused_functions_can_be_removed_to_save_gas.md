## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Unused functions can be removed to save gas](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/63) 

# Handle

SolidityScan


# Vulnerability details

### Description

Smart Contracts are Gas sensitive and heavily depend on how Gas is spent and managed across the code. This affects each and every function definition and logic. 

Therefore having any unused functions in the code cost unnecessary Gas usage and thus negatively impacts the Contract and the organization.

## Impact
Having unused function definitions and parameters negatively affects the contract and costs unnecessary Gas. This also makes it difficult and confusing for auditors to go through the code.

## Proof of Concept
The function "_getColls" is not used anywhere in the code

https://github.com/code-423n4/2021-12-yetifinance/blob/main/packages/contracts/contracts/Dependencies/LiquityBase.sol#L146-L153

## Tools Used

## Recommended Mitigation Steps
Evaluate if the function call should be used anywhere otherwise remove the function definition.

