## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- disagree with severity

# [Use of ecrecover is susceptible to signature malleability](https://github.com/code-423n4/2021-09-swivel-findings/issues/99) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The ecrecover function is used to verify and execute EIP-2612 permit transactions. The built-in EVM precompile ecrecover is susceptible to signature malleability (because of non-unique s and v values) which could lead to replay attacks (references: https://swcregistry.io/docs/SWC-117, https://swcregistry.io/docs/SWC-121 and https://medium.com/cryptronics/signature-replay-vulnerabilities-in-smart-contracts-3b6f7596df57). 

While this is not exploitable for replay attacks in the current implementation because of the use of nonces, this may become a vulnerability if used elsewhere.


## Proof of Concept

https://github.com/Swivel-Finance/gost/blob/5fb7ad62f1f3a962c7bf5348560fe88de0618bae/test/tokens/Erc2612.sol#L48

## Tools Used
Manual Analysis

## Recommended Mitigation Steps

Consider using OpenZeppelin’s ECDSA library (which prevents this malleability) instead of the built-in function: https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/cryptography/ECDSA.sol

