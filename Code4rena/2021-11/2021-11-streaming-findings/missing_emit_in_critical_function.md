## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Missing Emit in critical function](https://github.com/code-423n4/2021-11-streaming-findings/issues/35) 

# Handle

cyberboy


# Vulnerability details

## Impact
Events for critical state changes (e.g., owner and other critical parameters) should be emitted for tracking this off-chain. 

## Proof of Concept
https://github.com/code-423n4/2021-11-streaming/blob/main/Streaming/src/Locke.sol#L41-L43
The function "setEmergencyGov" is missing event emit, and it is a critical function used for setting emergency governer. 


## Tools Used
Slither

## Recommended Mitigation Steps
Add an event and emit it as a new emergency governor is set. 

