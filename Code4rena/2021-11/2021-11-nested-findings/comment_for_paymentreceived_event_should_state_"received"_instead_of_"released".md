## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Comment for PaymentReceived event should state "received" instead of "released"](https://github.com/code-423n4/2021-11-nested-findings/issues/44) 

# Handle

GiveMeTestEther


# Vulnerability details

## Impact
Comment in FeeSplitter  "/// @dev Emitted when a payment is released" for the PaymentReceived event should say "received" instead of "released".

## Proof of Concept
https://github.com/code-423n4/2021-11-nested/blob/5d113967cdf7c9ee29802e1ecb176c656386fe9b/contracts/FeeSplitter.sol#L25

## Tools Used
Manal Analysis

## Recommended Mitigation Steps
Change line to: /// @dev Emitted when a payment is received

