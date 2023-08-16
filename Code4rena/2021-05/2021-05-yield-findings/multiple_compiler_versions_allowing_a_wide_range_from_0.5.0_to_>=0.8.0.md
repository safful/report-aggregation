## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Multiple compiler versions allowing a wide range from 0.5.0 to >=0.8.0](https://github.com/code-423n4/2021-05-yield-findings/issues/54) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Project uses multiple compiler versions with most specifying ^0.8.0, some specifying >=0.8.0 which allows breaking versions >= 0.9.0 in future if reused/redeployed, and some even allowing much older >= 0.5.0/0.6.0. 

The dangers of allowing multiple compilers across breaking revisions is that the security bug fixes and features might be different across different contracts introducing vulnerabilities or giving a false sense of security.

For example, most contract use ^0.8.0 which means they have default checked arithmetic to prevent overflows/underflows without using OZ SafeMath. This doesn’t apply to the few (inherited) contracts that may be compiled with <0.8.0 and have unchecked overflows/underflows.


## Proof of Concept

^0.8.0: https://github.com/code-423n4/2021-05-yield/blob/e4c8491cd7bfa5dc1b59eb1b257161cd5bf8c6b0/contracts/Join.sol#L2

>= 0.8.0: https://github.com/code-423n4/2021-05-yield/blob/e4c8491cd7bfa5dc1b59eb1b257161cd5bf8c6b0/contracts/yieldspace/YieldMath.sol#L2

>= 0.5.0: https://github.com/code-423n4/2021-05-yield/blob/e4c8491cd7bfa5dc1b59eb1b257161cd5bf8c6b0/contracts/utils/token/SafeERC20Namer.sol#L3

>= 0.6.0: https://github.com/code-423n4/2021-05-yield/blob/e4c8491cd7bfa5dc1b59eb1b257161cd5bf8c6b0/contracts/utils/token/TransferHelper.sol#L4

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

1. Update all contracts to use pragma solidity ^0.8.0 or better a fixed version like 0.8.4
2. Deploy with the same compiler version which was used for testing

