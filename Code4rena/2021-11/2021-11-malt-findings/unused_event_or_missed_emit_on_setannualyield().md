## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Unused event or missed emit on SetAnnualYield()](https://github.com/code-423n4/2021-11-malt-findings/issues/69) 

# Handle

BouSalman


# Vulnerability details

## Vulnerability description 
The event **SetAnnualYield** on Contract **StabilizerNode** is defined but never emitted inside the Contract.

## Impact
Unused events in the codebase can be confusing, each declared event should have a corresponding emit statement.

## Proof of Concept
https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/StabilizerNode.sol#L73

## Tools Used
manual code review.

## Recommended Mitigation Steps
it's better to remove unused events from the code to improve coding quality, Also monitoring will be effected since no emit statements is there.

