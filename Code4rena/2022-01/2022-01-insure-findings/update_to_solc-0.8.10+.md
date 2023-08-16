## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Update to solc-0.8.10+](https://github.com/code-423n4/2022-01-insure-findings/issues/47) 

# Handle

TomFrenchBlockchain


# Vulnerability details

## Impact
Gas costs

## Proof of Concept

Solidity 0.8.10 has a useful change which reduced gas costs of external calls which expect a return value: https://blog.soliditylang.org/2021/11/09/solidity-0.8.10-release-announcement/

> Code Generator: Skip existence check for external contract if return data is expected. In this case, the ABI decoder will revert if the contract does not exist

InsureDAO is using 0.8.7: 
https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/Factory.sol#L8

Updating to the newer version of solc will allow InsureDAO to take advantage of these lower costs for external calls.

## Recommended Mitigation Steps

Update to solc 0.8.10 or above

