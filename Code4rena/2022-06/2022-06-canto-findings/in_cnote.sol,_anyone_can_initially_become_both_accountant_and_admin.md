## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [In Cnote.sol, anyone can initially become both accountant and admin](https://github.com/code-423n4/2022-06-canto-findings/issues/195) 

# Lines of code

https://github.com/Plex-Engineer/lending-market/blob/ab31a612be354e252d72faead63d86b844172761/contracts/CNote.sol#L14


# Vulnerability details

## Impact

Affected code:

- [https://github.com/Plex-Engineer/lending-market/blob/ab31a612be354e252d72faead63d86b844172761/contracts/CNote.sol#L14](https://github.com/Plex-Engineer/lending-market/blob/ab31a612be354e252d72faead63d86b844172761/contracts/CNote.sol#L14)

The function `_setAccountantContract()` is supposed to be called after contract initialization, so that the `accountant` is immediately set. However, this function completely lacks any access control (it’s just `public`) so an attacker can monitor the mempool and frontrun the transaction in order to become both `accountant` and `admin`

## Tools Used

Editor

## Recommended Mitigation Steps

The function should:

1. have a guard that regulates access control
2. not set the `admin` too, which is dangerous and out of scope

