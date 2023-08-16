## Tags

- bug
- 0 (Non-critical)
- resolved
- sponsor confirmed

# [Lack of 'emit' keyword in AlToken.sol](https://github.com/code-423n4/2021-11-yaxis-findings/issues/4) 

# Handle

tqts


# Vulnerability details

## Impact
An event is called without the emit keyword

## Proof of Concept
https://github.com/code-423n4/2021-11-yaxis/blob/0311dd421fb78f4f174aca034e8239d1e80075fe/contracts/v3/alchemix/AlToken.sol#L100

## Tools Used
Manual review

## Recommended Mitigation Steps
Add the 'emit' keyword in the event emission.

