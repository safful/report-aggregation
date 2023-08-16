## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- resolved

# [Inconsistent NatSpec comment in StakeLocker.sol](https://github.com/code-423n4/2021-04-maple-findings/issues/46) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Function _isValidPoolDelegate() is not about pause/unpause but about msg.sender being a valid Pool Delegate, which is used to check if msg.sender can set lockup period and open staking to public in StakeLocker.sol.

Therefore, the Natspec comment for this function is incorrect:
@dev Function to determine if msg.sender is eligible to trigger pause/unpause.


## Proof of Concept

https://github.com/maple-labs/maple-core/blob/355141befa89c7623150a83b7d56a5f5820819e9/contracts/StakeLocker.sol#L302-L307

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Change @dev Natspec comment to correctly indicate the functionality of _isValidPoolDelegate().


