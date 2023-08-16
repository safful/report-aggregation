## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- resolved

# [Camel case function name](https://github.com/code-423n4/2021-06-realitycards-findings/issues/13) 

# Handle

heiho1


# Vulnerability details

## Impact
Detailed description of the impact of this finding.

Minimal code quality issue.

## Proof of Concept
Provide direct links to all referenced code in GitHub. Add screenshots, logs, or any other relevant proof that illustrates the concept.

https://github.com/code-423n4/2021-06-realitycards/blob/86a816abb058cc0ed9b6f5c4a8ad146f22b8034c/contracts/interfaces/IRCFactory.sol#L26

The function setminimumPriceIncreasePercent does not follow the code standard of camel casing of function names.  

## Tools Used

Manual code review.

## Recommended Mitigation Steps

Rename the function to have proper camel casing.

