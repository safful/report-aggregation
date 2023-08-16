## Tags

- bug
- resolved
- sponsor confirmed
- QA (Quality Assurance)

# [QA Report](https://github.com/code-423n4/2022-03-paladin-findings/issues/36) 

# Lines of code

https://github.com/code-423n4/2022-03-paladin/blob/main/contracts/HolyPaladinToken.sol#L729


# Vulnerability details

## Impact
If `startDropPerSecond` is initialized at less than `endDropPerSecond` the contract will be unusable. There will be an underflow in `_updateDropPerSecond` which will always revert. This function is called throughout the contract, in critical functions like `lock` and `claim`, if it were to always revert the contract would be broken and unusable.

## Proof of Concept
If `startDropPerSecond` is initialized at less than `endDropPerSecond` in the constructor, the contract will be deployed without issue but will be broken.

## Tools Used
Manual analysis

## Recommended Mitigation Steps
Add a check in the constructor that ensures `startDropPerSecond` >= `endDropPerSecond`

