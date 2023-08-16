## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Function cooldown() is not protected when protocol in emergency mode](https://github.com/code-423n4/2022-03-paladin-findings/issues/54) 

# Lines of code

https://github.com/code-423n4/2022-03-paladin/blob/9c26ec8556298fb1dc3cf71f471aadad3a5c74a0/contracts/HolyPaladinToken.sol#L228-L235


# Vulnerability details

Function cooldown() is not protected when protocol is in emergency mode.
Its behavior is not consistent with the other major functions defined.

## Impact
While other major functions like stake, unstake, lock, unlock, etc., of this contract is protected by checking for emergency flag and reverting, 
this function cooldown() is not checked. The impact of this is that during emergency mode, users can set immediately the cooldown() and plan for unstaking when the emergency mode is lifted and cooldown period expires. This may not be the desirable behaviour expected by the protocol.

## Proof of Concept
Contract Name : HolyPaladinToken.sol
Function cooldown()

## Recommended Mitigation Steps
Add checking for emergency mode for this function also.
```
if(emergency) revert EmergencyBlock();
```


