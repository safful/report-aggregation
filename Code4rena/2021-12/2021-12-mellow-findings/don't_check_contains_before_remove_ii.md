## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Don't check contains before remove II](https://github.com/code-423n4/2021-12-mellow-findings/issues/26) 

# Handle

0x1f8b


# Vulnerability details

# Vulnerability details

## Impact
Gas optimization.

## Proof of Concept
The method remove of the library AddressSet doesn't fail if the entry was not found (https://github.com/OpenZeppelin/openzeppelin-contracts/blob/a05312f1b72acca6904ffe32ef83ccdbad20cb4f/contracts/utils/structs/EnumerableSet.sol#L72), this method return true or false if was removed, so it's not needed to check if _vaultGovernances.contains(addr) in the method removeFromVaultGovernances from ProtocolGovernance contract.

## Tools Used
Manual review

## Recommended Mitigation Steps
Remove the contains conditional

