## Tags

- bug
- 0 (Non-critical)
- resolved
- sponsor confirmed

# [Convert Emergency Exit Check to Modifier.](https://github.com/code-423n4/2021-11-yaxis-findings/issues/11) 

# Handle

TimmyToes


# Vulnerability details

## Impact
Gas saving on deployment.
Guaranteed consistency, especially if making the same check across multiple functions (and I'm about to suggest that you might want to check this more).
Increased functionality of inheriting contracts.
Improved readability and code organisation.
Basically, every reason that modifiers exist in the first place.

## Proof of Concept
https://github.com/code-423n4/2021-11-yaxis/blob/0311dd421fb78f4f174aca034e8239d1e80075fe/contracts/v3/alchemix/Alchemist.sol#L457
https://github.com/code-423n4/2021-11-yaxis/blob/0311dd421fb78f4f174aca034e8239d1e80075fe/contracts/v3/alchemix/Alchemist.sol#L489

## Tools Used

## Recommended Mitigation Steps
Convert emergency exit check to modifier.


