## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [reassignGlobalAdmin() Have No Transfer Ownership Pattern](https://github.com/code-423n4/2021-11-malt-findings/issues/112) 

# Handle

Meta0xNull


# Vulnerability details

## Impact
The current ownership transfer process involves the current TIMELOCK_ROLE calling reassignGlobalAdmin().  If the nominated EOA account is not a valid account, it is entirely possible the owner may accidentally transfer ownership to an uncontrolled account, breaking all functions with the TIMELOCK_ROLE modifier.

## Proof of Concept
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/Permissions.sol#L63-L77

## Tools Used
Manual Review

## Recommended Mitigation Steps
Consider implementing a two step process where the TIMELOCK_ROLE nominates an account and the nominated account needs to call an accept_TIMELOCK_ROLE() function for the transfer of ownership to fully succeed. This ensures the nominated EOA account is a valid and active account.


