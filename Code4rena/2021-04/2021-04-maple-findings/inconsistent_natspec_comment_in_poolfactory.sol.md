## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- resolved

# [Inconsistent NatSpec comment in PoolFactory.sol](https://github.com/code-423n4/2021-04-maple-findings/issues/58) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Function whenProtocolNotPaused() is not about msg.sender eligibility to pause/unpause but about checking if protocol is unpaused or not in PoolFactory.sol.

Therefore, the Natspec comment for this function is incorrect:
@dev Function to determine if msg.sender is eligible to trigger pause/unpause.


## Proof of Concept

https://github.com/maple-labs/maple-core/blob/355141befa89c7623150a83b7d56a5f5820819e9/contracts/PoolFactory.sol#L152-L157

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Change @dev Natspec comment to correctly indicate the functionality of _whenProtocolNotPaused().


