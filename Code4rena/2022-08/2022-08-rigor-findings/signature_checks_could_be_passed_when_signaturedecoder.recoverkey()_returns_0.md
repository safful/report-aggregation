## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- valid

# [Signature Checks could be passed when SignatureDecoder.recoverKey() returns 0](https://github.com/code-423n4/2022-08-rigor-findings/issues/179) 

# Lines of code

https://github.com/code-423n4/2022-08-rigor/blob/main/contracts/Project.sol#L887
https://github.com/code-423n4/2022-08-rigor/blob/main/contracts/Project.sol#L108-L115


# Vulnerability details

## Impact
It is possible to pass Signature Validity check with an SignatureDecoder.recoverKey() returns 0 whenever the builder and /or contractor have an existing approved hash for a data.

With occurrence of above, any user can call changeOrder or setComplete functions successfully after  user approves data hashes.


## Tools Used
Manual review

## Recommended Mitigation Steps
There should be a require check for `_recoveredSignature != 0` in checkSignatureValidity()