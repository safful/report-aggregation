## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Consider making some constants as non-public to save gas](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/95) 

# Handle

Jujic


# Vulnerability details

## Impact
Each function part of contract's external interface is part of the function dispatch, i.e., every time a contract is called, it goes through a switch statement (a set of eq ... JUMPI blocks in EVM)
matching the selector of each externally available functions with the chosen function selector (the first 4 bytes of calldata).
This means that any unnecessary function that is part of contract's external interface will lead to more gas for (almost) every single function calls to the contract. There are several cases where constants were made public. This is unnecessary; the constants can simply be readfrom the verified contract, i.e., it is unnecessary to expose it with a public function.

## Proof of Concept
https://github.com/code-423n4/2021-12-yetifinance/blob/5f5bf61209b722ba568623d8446111b1ea5cb61c/packages/contracts/contracts/TroveManagerRedemptions.sol#L53-L55

https://github.com/code-423n4/2021-12-yetifinance/blob/5f5bf61209b722ba568623d8446111b1ea5cb61c/packages/contracts/contracts/TroveManager.sol#L35-L39


Example:
```
uint256 public constant BOOTSTRAP_PERIOD = 14 days;
```
## Tools Used
Remix
## Recommended Mitigation Steps

