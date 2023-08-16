## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Use of constant `keccak` variables results in extra hashing (and so gas).](https://github.com/code-423n4/2021-10-slingshot-findings/issues/3) 

# Handle

TomFrench


# Vulnerability details

## Impact
Increase gas costs on all `onlyAdmin` operations

## Proof of Concept

The `SLINGSHOT_ADMIN_ROLE` variable is marked as `constant`:
https://github.com/code-423n4/2021-10-slingshot/blob/f6e7a0a39e3267bbe3c7fe60d6074cbf54f5750f/contracts/Adminable.sol#L11

This results in the `keccak` operation being performed whenever the variable is used, increasing gas costs relative to just storing the output hash. Changing to `immutable` will only perform hashing on contract deployment which will save gas.

See: https://github.com/ethereum/solidity/issues/9232#issuecomment-646131646

## Recommended Mitigation Steps

Change the variable to be `immutable` rather than `constant`

