## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- disagree with severity

# [Pool.burnSynth(address,address) is potentially reentrant](https://github.com/code-423n4/2021-07-spartan-findings/issues/203) 

# Handle

heiho1


# Vulnerability details

## Impact

Pool.burnSynth(address,address) is potentially a reentrant method because it executes transfers and burning before updating balances/metrics.

## Proof of Concept

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Pool.sol#L245

## Tools Used

Slither

## Recommended Mitigation Steps

The function should update state before external calls.

Consider using a nonReentrant guard as provided by OpenZeppelin:

https://docs.openzeppelin.com/contracts/4.x/api/security#ReentrancyGuard

