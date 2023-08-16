## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Wrong logic around `areOperatorsImported`](https://github.com/code-423n4/2022-02-nested-findings/issues/17) 

# Lines of code

https://github.com/code-423n4/2022-02-nested/blob/fe6f9ef7783c3c84798c8ab5fc58085a55cebcfc/contracts/OperatorResolver.sol#L42-L43


# Vulnerability details

## Impact
The logic related to the `areOperatorsImported` method is incorrect and can cause an operator not to be updated because the owner thinks it is already updated, and a vulnerable or defective one can be used.

## Proof of Concept
The `operators` mapping is made up of a key `bytes32 name` and a value made up of two values: `implementation` and `selector`, both of which identify the contract and function to be called when an operator is invoked.

The `areOperatorsImported` method tries to check if the operators to check already exist, however, the check is not done correctly, since && is used instead of ||.

If the operator with name `A` and value `{implementation=0x27f8d03b3a2196956ed754badc28d73be8830a6e,selector="performSwapVulnerable"}` exists, and the owner try to check if the operator with name `A` and value `{implementation=0x27f8d03b3a2196956ed754badc28d73be8830a6e,selector="performSwapFixed"}` exists, that function will return `true`, and the owner may decide not to import it , producing unexpected errors.
Because operators manage the tokens, this error can produce a token lost.

## Recommended Mitigation Steps
Change && by || 


