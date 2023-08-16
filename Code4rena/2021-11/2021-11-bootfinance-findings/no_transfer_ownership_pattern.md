## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [No Transfer Ownership Pattern](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/35) 

# Handle

defsec


# Vulnerability details

## Impact

The current ownership transfer process involves the current owner calling Swap.transferOwnership(). This function checks the new owner is not the zero address and proceeds to write the new owner's address into the owner's state variable. If the nominated EOA account is not a valid account, it is entirely possible the owner may accidentally transfer ownership to an uncontrolled account, breaking all functions with the onlyOwner() modifier.

## Proof of Concept

1. Navigate to "https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/customswap/contracts/Swap.sol#L30"
2. The contract has many onlyOwner function.
3. The contract is inherited from the Ownable which includes transferOwnership.

## Tools Used

None

## Recommended Mitigation Steps

Implement zero address check and Consider implementing a two step process where the owner nominates an account and the nominated account needs to call an acceptOwnership() function for the transfer of ownership to fully succeed. This ensures the nominated EOA account is a valid and active account.

