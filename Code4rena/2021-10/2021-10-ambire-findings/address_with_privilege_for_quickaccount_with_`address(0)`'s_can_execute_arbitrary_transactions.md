## Tags

- bug
- sponsor confirmed
- disagree with severity
- 1 (Low Risk)
- resolved

# [Address with privilege for QuickAccount with `address(0)`'s can execute arbitrary transactions](https://github.com/code-423n4/2021-10-ambire-findings/issues/3) 

# Handle

pmerkleplant


# Vulnerability details

## Impact
If a caller has privileges for a QuickAccount consisting of two `address(0)`'s,
then the caller can execute arbitrary transactions through the 
`QuicAccManager::send()` function.

## Proof of Concept
A caller of the `QuickAccManager::send()` needs to be privileged for the 
QuickAccount the caller provides as argument ([line 57](https://github.com/code-423n4/2021-10-ambire/blob/main/contracts/wallet/QuickAccManager.sol#L57)).

As an arbitrary value can be set as `privileged[caller]` in `Indentity.sol`, so
can a QuickAccount struct consisting of two `address(0)`'s.

The following calls to `SignatureValidator::recoverAddr()` (line 69, 70 and 73)
can be made to always return `address(0)` if the signature has 
`SignatureMode.NoSig`.

As the signature is provided as argument in `QuickAccManager::send()`, the 
caller has total control of it.

The checks in line [69, 70](https://github.com/code-423n4/2021-10-ambire/blob/main/contracts/wallet/QuickAccManager.sol#L69)
and [73](https://github.com/code-423n4/2021-10-ambire/blob/main/contracts/wallet/QuickAccManager.sol#L73)
now always pass as long as the accounts in the QuickAccount struct are 
`address(0)` too.

Therefore, a caller with permissions for such a QuickAccount can execute and 
schedule arbitrary transactions without the need for valid signatures.

## Tools Used
-

## Recommended Mitigation Steps
Add a check in the `QuickAccManager::send()` function to forbid 
QuickAccounts with `address(0)`.

