## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Delete - ABI Coder V2 For Gas Optimization](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/89) 

# Handle

defsec


# Vulnerability details

## Impact

From Pragma 0.8.0, ABI coder v2 is activated by default. The pragma abicoder v2 can be deleted from the repository. That will provide gas optimization.

## Proof of Concept

1. The following contract is using ABI coder v2.

"https://github.com/code-423n4/2021-12-yetifinance/blob/1da782328ce4067f9654c3594a34014b0329130a/packages/contracts/contracts/YETI/sYETIToken.sol#L3"


## Tools Used

None

## Recommended Mitigation Steps

Upgrade pragma to 0.8.0  and After the 0.8.0, ABI coder v2 is activated by default. Upgrade pragma to 0.8.0 version. It is recommended to delete redundant codes.

From Solidity v0.8.0 Breaking Changes https://docs.soliditylang.org/en/v0.8.0/080-breaking-changes.html

