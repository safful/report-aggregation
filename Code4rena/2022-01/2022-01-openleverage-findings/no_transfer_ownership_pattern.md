## Tags

- bug
- 0 (Non-critical)
- resolved
- sponsor confirmed

# [No Transfer Ownership Pattern](https://github.com/code-423n4/2022-01-openleverage-findings/issues/65) 

# Handle

cccz


# Vulnerability details

## Impact
The current ownership transfer process involves the current owner calling transferOwnership(). This function checks the new owner is not the zero address and proceeds to write the new owner’s address into the owner’s state variable. If the nominated EOA account is not a valid account, it is entirely possible the owner may accidentally transfer ownership to an uncontrolled account, breaking all functions with the onlyOwner() modifier.

## Proof of Concept
https://github.com/code-423n4/2022-01-openleverage/blob/main/openleverage-contracts/contracts/Airdrop.sol#L9

## Tools Used

None

## Recommended Mitigation Steps

Implement zero address check and consider implementing a two step process where the owner nominates an account and the nominated account needs to call an acceptOwnership() function for the transfer of ownership to fully succeed. This ensures the nominated EOA account is a valid and active account.


