## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Learn from the past](https://github.com/code-423n4/2021-12-mellow-findings/issues/20) 

# Handle

0x1f8b


# Vulnerability details

## Impact
Mandatory check that could produce undesired results.

## Proof of Concept
The smart contract ChiefTrader was in charge of the swaps, and the method _requireAllowedTokens is in charge to know that all paths are valid, it's mandatory to check that token0 and token1 are not equal, you can see a previous hack in the following link, where the hacker use the same from and to for change the price of the token https://twitter.com/mudit__gupta/status/1465726874974187524?s=12 .

## Tools Used
Manual review

## Recommended Mitigation Steps
Add require for check that token0 and token1 are different.

