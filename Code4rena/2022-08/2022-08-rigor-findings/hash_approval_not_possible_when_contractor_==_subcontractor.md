## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- valid

# [Hash approval not possible when contractor == subcontractor](https://github.com/code-423n4/2022-08-rigor-findings/issues/86) 

# Lines of code

https://github.com/code-423n4/2022-08-rigor/blob/f2498c86dbd0e265f82ec76d9ec576442e896a87/contracts/Project.sol#L859


# Vulnerability details

## Impact & Proof Of Concept
When a contractor (let's say Bob) is also a subcontractor (which can be a valid scenario), it is not possible to use the hash approval feature for `checkSignatureTask`. The first call to `checkSignatureValidity` will already delete `approvedHashes[address(Bob)][_hash]`, the second call therefore fails.

Note that the same situation would also be possible for builder == contractor, or builder == subcontractor, although those situations are probably less likely to occur.

## Recommended Mitigation Steps
Delete the approval only when all checks are done.