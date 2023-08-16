## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Useless imports](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/72) 

# Handle

Jujic


# Vulnerability details

## Impact
contract WJLP does not need to import this contract in production
```
import "hardhat/console.sol";
```

## Proof of Concept
https://github.com/code-423n4/2021-12-yetifinance/blob/5f5bf61209b722ba568623d8446111b1ea5cb61c/packages/contracts/contracts/AssetWrappers/WJLP/WJLP.sol#L8

https://github.com/code-423n4/2021-12-yetifinance/blob/5f5bf61209b722ba568623d8446111b1ea5cb61c/packages/contracts/contracts/AssetWrappers/WJLP/WJLP.sol#L152-L160

## Tools Used
Remix

## Recommended Mitigation Steps
Remove this contract  to reduce the size of the contract and thus save some deployment gas.

