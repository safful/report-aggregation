## Tags

- bug
- QA (Quality Assurance)
- sponsor confirmed

# [QA Report](https://github.com/code-423n4/2022-03-rolla-findings/issues/63) 

## Title 1
Missing input validation on array lengths

### Impact
The functions below fail to perform input validation on arrays to verify the lengths match. A mismatch could lead to an exception or undefined behavior.

## #Proof of Concept
https://github.com/RollaProject/quant-protocol/blob/main/contracts/options/CollateralToken.sol#L138-L160

https://github.com/RollaProject/quant-protocol/blob/main/contracts/options/CollateralToken.sol#L163-L184

### Tools Used
Manual review on VScode

### Recommended Mitigation Steps
Add input validation to check that the length of both arrays match.

---------------------------------------------------------------------------------------------------------------------

--------------------------------------------------------------------------------------------------------------------
## Title 2
Use of ecrecover is susceptible to signature malleability

### Impact
The ecrecover function is used in metaSetApprovalForAll() to recover the address from the signature. The built-in EVM precompile ecrecover is susceptible to signature malleability which could lead to replay attacks (references: https://swcregistry.io/docs/SWC-117, https://swcregistry.io/docs/SWC-121 and https://medium.com/cryptronics/signature-replay-vulnerabilities-in-smart-contracts-3b6f7596df57).


## #Proof of Concept
https://github.com/RollaProject/quant-protocol/blob/main/contracts/options/CollateralToken.sol#L218-L219

### Tools Used
Manual review on VScode

### Recommended Mitigation Steps
Consider using the openzeppelin ECDSA library for signature verifications.