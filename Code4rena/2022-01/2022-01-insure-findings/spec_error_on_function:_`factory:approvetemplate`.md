## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [Spec error on function: `Factory:approveTemplate`](https://github.com/code-423n4/2022-01-insure-findings/issues/296) 

# Handle

Dravee


# Vulnerability details

The spec doesn't match with the comments in the code here: 

Code: https://github.com/code-423n4/2022-01-insure/blob/main/contracts/Factory.sol#L90-L91
Spec: https://insuredao.gitbook.io/developers/market/factory#approvetemplate

Here, the spec doesn't mention `_isOpen` and seem to confuse the `_approval` description with what `_isOpen` should be.

## Tools Used
VS Code

## Recommended Mitigation Steps
My guess is that the spec should be corrected

