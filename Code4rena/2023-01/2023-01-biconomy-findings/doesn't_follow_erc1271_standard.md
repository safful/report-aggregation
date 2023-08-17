## Tags

- bug
- 2 (Med Risk)
- judge review requested
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-06

# [Doesn't Follow ERC1271 Standard](https://github.com/code-423n4/2023-01-biconomy-findings/issues/288) 

# Lines of code

https://github.com/code-423n4/2023-01-biconomy/blob/main/scw-contracts/contracts/smart-contract-wallet/interfaces/ISignatureValidator.sol#L6
https://github.com/code-423n4/2023-01-biconomy/blob/main/scw-contracts/contracts/smart-contract-wallet/interfaces/ISignatureValidator.sol#L19
https://github.com/code-423n4/2023-01-biconomy/blob/main/scw-contracts/contracts/smart-contract-wallet/SmartAccount.sol#L342


# Vulnerability details

## Impact

As Per [EIP-1271](https://eips.ethereum.org/EIPS/eip-1271) standard `ERC1271_MAGIC_VAULE` should be `0x1626ba7e` instead of `0x20c13b0b` and function name should be `isValidSignature(bytes32,bytes)` instead of  `isValidSignature(bytes,bytes)`. Due to this, signature verifier contract go fallback function and return unexpected value and never return `ERC1271_MAGIC_VALUE` and always revert `execTransaction` function. 

## Proof of Concept

https://github.com/code-423n4/2023-01-biconomy/blob/main/scw-contracts/contracts/smart-contract-wallet/interfaces/ISignatureValidator.sol#L6

https://github.com/code-423n4/2023-01-biconomy/blob/main/scw-contracts/contracts/smart-contract-wallet/interfaces/ISignatureValidator.sol#L19

https://github.com/code-423n4/2023-01-biconomy/blob/main/scw-contracts/contracts/smart-contract-wallet/SmartAccount.sol#L342


## Tools Used
Manual Review

## Recommended Mitigation Steps
Follow [EIP-1271](https://eips.ethereum.org/EIPS/eip-1271) standard. 