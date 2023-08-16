## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Incorrect number of seconds in `ONE_YEAR` variable](https://github.com/code-423n4/2022-03-paladin-findings/issues/4) 

# Lines of code

https://github.com/code-423n4/2022-03-paladin/blob/main/contracts/HolyPaladinToken.sol#L25


# Vulnerability details

## Impact
In `HolyPaladinToken.sol` the `ONE_YEAR` variable claims that there are `31557600` seconds in a year when this is incorrect.  The `ONE_YEAR` variable is used in the `getCurrentVotes()` function as well as the `getPastVotes()` function so it is vital that the correct time in seconds be used as it can effect users negatively. 

## Proof of Concept
https://github.com/code-423n4/2022-03-paladin/blob/main/contracts/HolyPaladinToken.sol#L25

86,400 seconds in a day x 365 = 31_536_000

## Tools Used
Manual code review 

## Recommended Mitigation Steps
The correct number of seconds in a year is 31_536_000 so the `ONE_YEAR` variable should be changed to `ONE_YEAR = 31_536_000` 

