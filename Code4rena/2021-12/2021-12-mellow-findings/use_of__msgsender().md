## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Use of _msgSender()](https://github.com/code-423n4/2021-12-mellow-findings/issues/81) 

# Handle

defsec


# Vulnerability details

## Impact

The use of _msgSender() when there is no implementation of a meta transaction mechanism that uses it, such as EIP-2771, very slightly increases gas consumption.


## Proof of Concept

_msgSender() is utilized three times where msg.sender could have been used in the following function.


"""
https://github.com/code-423n4/2021-12-mellow/blob/main/mellow-vaults/contracts/VaultRegistry.sol#L96

https://github.com/code-423n4/2021-12-mellow/blob/main/mellow-vaults/contracts/VaultRegistry.sol#L106

https://github.com/code-423n4/2021-12-mellow/blob/main/mellow-vaults/contracts/VaultRegistry.sol#L88

"""


## Tools Used

None

## Recommended Mitigation Steps

Replace _msgSender() with msg.sender if there is no mechanism to support meta-transactions like EIP-2771 implemented.

