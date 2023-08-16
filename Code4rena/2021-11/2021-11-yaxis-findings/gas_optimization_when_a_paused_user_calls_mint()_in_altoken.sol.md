## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas optimization when a paused user calls mint() in AlToken.sol](https://github.com/code-423n4/2021-11-yaxis-findings/issues/2) 

# Handle

tqts


# Vulnerability details

## Impact
Gas saved when a paused user calls mint()

## Proof of Concept
https://github.com/code-423n4/2021-11-yaxis/blob/0311dd421fb78f4f174aca034e8239d1e80075fe/contracts/v3/alchemix/AlToken.sol#L68

## Tools Used
Manual review

## Recommended Mitigation Steps
Check for the paused condition before checking for the ceiling condition. If the user is paused, the function reverts earlier, saving gas.

