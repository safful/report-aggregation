## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [UserLock information can be found during emergency mode](https://github.com/code-423n4/2022-03-paladin-findings/issues/59) 

# Lines of code

https://github.com/code-423n4/2022-03-paladin/blob/9c26ec8556298fb1dc3cf71f471aadad3a5c74a0/contracts/HolyPaladinToken.sol#L446-L468


# Vulnerability details


When the contract is in blocked state (emergency mode), the protocol wants to return an empty UserLock info, on calling the function getUserLock.
However, there is another way, by which the users can find the same information.

The below function is not protected when in emergency mode, and users can use this alternatively.
Line#466 function getUserPastLock(address user, uint256 blockNumber) 

## Impact
There is no loss of funds, however the intention to block information (return empty lock info) is defeated, because not all functions are protected.
There is inconsistency in implementing the emergency mode check.

## Proof of Concept
Contract Name : HolyPaladinToken.sol
Functions getUserLock and getUserPastLock

## Recommended Mitigation Steps
Add checking for emergency mode for this function getUserPastLock.
```
if(emergency) revert EmergencyBlock();
```
Additional user access check can be added, so that the function returns correct value when the caller(msg.sender) is admin or owner.


