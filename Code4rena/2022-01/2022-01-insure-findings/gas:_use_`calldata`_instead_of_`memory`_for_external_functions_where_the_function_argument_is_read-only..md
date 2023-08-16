## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas: Use `calldata` instead of `memory` for external functions where the function argument is read-only.](https://github.com/code-423n4/2022-01-insure-findings/issues/64) 

# Handle

Dravee


# Vulnerability details

## Impact  
On external functions, when using the `memory` keyword with a function argument, what's happening is that a `memory` acts as an intermediate.  
  
Reading directly from `calldata` using `calldataload` instead of going via `memory` saves the gas from the intermediate memory operations that carry the values.  
  
As an extract from https://ethereum.stackexchange.com/questions/74442/when-should-i-use-calldata-and-when-should-i-use-memory :  
> `memory` and `calldata` (as well as `storage`) are keywords that define the data area where a variable is stored. To answer your question directly, `memory` should be used when declaring variables (both function parameters as well as inside the logic of a function) that you want stored in memory (temporary), and `calldata` _must_ be used when declaring an **external** function's **dynamic** parameters. The easiest way to think about the difference is that `calldata` is a non-modifiable, non-persistent area where function arguments are stored, and behaves mostly like memory.  
  
## Proof of Concept  
```  
Vault.sol:92:        address[2] memory _beneficiaries,
Vault.sol:93:        uint256[2] memory _shares
```  
  
## Tools Used  
VS Code  
  
## Recommended Mitigation Steps  
Use `calldata` instead of `memory` for external functions where the function argument is read-only.


