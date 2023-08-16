## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [withdrawAVAX() function has call to sender without reentrancy protection ](https://github.com/code-423n4/2022-01-trader-joe-findings/issues/32) 

# Handle

jayjonah8


# Vulnerability details

## Impact
In LauchEvent.sol the withdrawAVAX() function makes an external call to the msg.sender by way of _safeTransferAVAX.  This allows the caller to reenter this and other functions in this and other protocol files.  To prevent reentrancy and cross function reentrancy there should be reentrancy guard modifiers placed on the withdrawAVAX() function and any other function that makes external calls to the caller. 

## Proof of Concept
https://github.com/code-423n4/2022-01-trader-joe/blob/main/contracts/LaunchEvent.sol#L368

https://github.com/code-423n4/2022-01-trader-joe/blob/main/contracts/LaunchEvent.sol#L370

## Tools Used
Manual code review 

## Recommended Mitigation Steps
Add reentrancy guard modifier to withdrawAVAX() function. 

