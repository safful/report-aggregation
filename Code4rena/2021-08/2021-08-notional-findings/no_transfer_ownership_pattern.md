## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [No Transfer Ownership Pattern](https://github.com/code-423n4/2021-08-notional-findings/issues/94) 

# Handle

leastwood


# Vulnerability details

## Impact

The current ownership transfer process involves the current owner calling `NoteERC20.transferOwnership()`. This function checks the new owner is not the zero address and proceeds to write the new owner's address into the owner's state variable. If the nominated EOA account is not a valid account, it is entirely possible the owner may accidentally transfer ownership to an uncontrolled account, breaking all functions with the `onlyOwner()` modifier.

## Proof of Concept

https://github.com/code-423n4/2021-08-notional/blob/main/contracts/external/governance/NoteERC20.sol#L123-L127

## Tools Used

Manual code review

## Recommended Mitigation Steps

Consider implementing a two step process where the owner nominates an account and the nominated account needs to call an `acceptOwnership()` function for the transfer of ownership to fully succeed. This ensures the nominated EOA account is a valid and active account.

