## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Redundant Code Statement](https://github.com/code-423n4/2021-10-slingshot-findings/issues/27) 

# Handle

defsec


# Vulnerability details

## Impact

From Pragma 0.8.0, ABI coder v2 is activated by default. The pragma abicoder v2 can be deleted from the repository. That will provide gas optimization.

## Proof of Concept

1. Navigate to the following code sections.


https://github.com/code-423n4/2021-10-slingshot/blob/main/contracts/Adminable.sol#L3

https://github.com/code-423n4/2021-10-slingshot/blob/main/contracts/ApprovalHandler.sol#L3

https://github.com/code-423n4/2021-10-slingshot/blob/main/contracts/Executioner.sol#L3

https://github.com/code-423n4/2021-10-slingshot/blob/main/contracts/Slingshot.sol#L3

## Tools Used

None

## Recommended Mitigation Steps

ABI coder v2 is activated by default. It is recommended to delete redundant codes. 

From Solidity v0.8.0 Breaking Changes https://docs.soliditylang.org/en/v0.8.0/080-breaking-changes.html

