## Tags

- bug
- disagree with severity
- QA (Quality Assurance)
- resolved
- sponsor confirmed

# [Duplicate Contract Names](https://github.com/code-423n4/2022-05-aura-findings/issues/12) 

# Lines of code

https://github.com/code-423n4/2022-05-aura/blob/main/contracts/CrvDepositorWrapper.sol#L9
https://github.com/code-423n4/2022-05-aura/blob/main/contracts/AuraStakingProxy.sol#L10


# Vulnerability details

## Impact
If a codebase has two contracts with the same names, the compilation artifacts will not contain one of the contracts.

`ICrvDepositor` exists in both `AuraStakingProxy` and `CrvDepositorWrapper`

## Tools
Manual Review
 
## Recommended Mitigation Steps
Move the contract to an interface file and import it or if the interface differs rename one of the contracts.

