## Tags

- bug
- sponsor confirmed
- 0 (Non-critical)

# [Different pragma solidity](https://github.com/code-423n4/2021-04-vader-findings/issues/25) 

# Handle

gpersoon


# Vulnerability details

## Impact
Vault.sol has a different pragma statement than the rest, it contains an additional "^".

For the record the Vether.sol contract (as deployed here https://etherscan.io/address/0x4Ba6dDd7b89ed838FEd25d208D4f644106E34279#code), 
has a different solidity version.

It's cleaner to use the same versions.

## Proof of Concept

DAO.sol:pragma solidity 0.8.3;
Factory.sol:pragma solidity 0.8.3;
Pools.sol:pragma solidity 0.8.3;
Router.sol:pragma solidity 0.8.3;
Synth.sol:pragma solidity 0.8.3;
USDV.sol:pragma solidity 0.8.3;
Utils.sol:pragma solidity 0.8.3;
Vader.sol:pragma solidity 0.8.3;
Vault.sol:pragma solidity ^0.8.3;
Vether.sol:pragma solidity 0.6.4;

## Tools Used
Editor

## Recommended Mitigation Steps
Use the same solidity versions


