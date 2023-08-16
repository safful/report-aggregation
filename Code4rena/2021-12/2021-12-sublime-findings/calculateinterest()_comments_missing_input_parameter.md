## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [calculateInterest() comments missing input parameter](https://github.com/code-423n4/2021-12-sublime-findings/issues/87) 

# Handle

sirhashalot


# Vulnerability details

## Impact

The public `calculateInterest` function in CreditLine.sol is missing a @param comment for the `_timeElapsed` parameter. This parameter is obviously important and the units should be clearly stated as seconds.

## Proof of Concept

The `calculateInterest` function in CreditLine.sol:
https://github.com/code-423n4/2021-12-sublime/blob/main/contracts/CreditLine/CreditLine.sol#L385-L395 

## Tools Used

Manual analysis

## Recommended Mitigation Steps

Add the following line to the `calculateInterest` function comments in CreditLine.sol:
`* @param _timeElapsed Seconds elapsed since lastPrincipalUpdateTime`

