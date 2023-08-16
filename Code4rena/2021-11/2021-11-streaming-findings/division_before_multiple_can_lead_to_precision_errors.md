## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Division before multiple can lead to precision errors](https://github.com/code-423n4/2021-11-streaming-findings/issues/28) 

# Handle

cyberboy


# Vulnerability details

## Impact
Performing multiplication before division is generally better to avoid loss of precision because Solidity integer division might truncate

## Proof of Concept
https://github.com/code-423n4/2021-11-streaming/blob/main/Streaming/src/Locke.sol#L237-L238
globalStreamingSpeedPerSecond is later used for unstreamed for multiplication after performing division while calculation of globalStreamingSpeedPerSecond


## Tools Used
Slither 

## Recommended Mitigation Steps
The code can be optimized to use
uint112((uint256(tdelta) * (uint256(unstreamed) * 10**6) / (endStream - lastUpdate) * 10**6
Or maybe just 
(uint112((uint256(tdelta) * (uint256(unstreamed)) / (endStream - lastUpdate) 



