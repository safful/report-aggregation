## Tags

- bug
- 1 (Low Risk)
- sponsor acknowledged
- sponsor confirmed

# [Compact signatures not being supported could lead to DoS](https://github.com/code-423n4/2021-09-swivel-findings/issues/104) 

# Handle

0xRajeev


# Vulnerability details

## Impact

This implementation of Sig.sol doesn’t support compact signature (EIP-2098), where signature length can be 64 bytes instead of 65, as supported in the widely used OpenZeppelin’s ECDSA library. This lack of support could lead to DoS for users/clients that use compact signatures.

## Proof of Concept

https://github.com/Swivel-Finance/gost/blob/5fb7ad62f1f3a962c7bf5348560fe88de0618bae/test/swivel/Sig.sol#L41

See https://github.com/OpenZeppelin/openzeppelin-contracts/blob/1b27c13096d6e4389d62e7b0766a1db53fbb3f1b/contracts/utils/cryptography/ECDSA.sol#L57

https://eips.ethereum.org/EIPS/eip-2098

## Tools Used
Manual Analysis

## Recommended Mitigation Steps
Consider adding support for compact signatures, use OZ ECDSA library or highlight in documentation about this lack of support for EIP-2098.

