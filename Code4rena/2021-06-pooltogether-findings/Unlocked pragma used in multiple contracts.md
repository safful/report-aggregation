## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Unlocked pragma used in multiple contracts](https://github.com/code-423n4/2021-06-pooltogether-findings/issues/109) 

# Handle

shw


# Vulnerability details

## Impact

Some contracts (e.g., `PrizePool`) use an unlocked pragma (e.g., `pragma solidity >=0.6.0 <0.7.0;`) which is not fixed to a specific Solidity version. Locking the pragma helps ensure that contracts do not accidentally get deployed using a different compiler version with which they have been tested the most.

## Proof of Concept

Referenced code:
Please use `grep -R pragma .` to find the unlocked pragma statements.

## Recommended Mitigation Steps

Lock pragmas to a specific Solidity version. Consider the compiler bugs in the following lists and ensure the contracts are not affected by them. It is also recommended to use the latest version of Solidity when deploying contracts (see [Solidity docs](https://docs.soliditylang.org/en/v0.8.4/#solidity)).

Solidity compiler bugs:
[Solidity repo - known bugs](https://github.com/ethereum/solidity/blob/develop/docs/bugs.json)
[Solidity repo - bugs by version](https://github.com/ethereum/solidity/blob/develop/docs/bugs_by_version.json)

