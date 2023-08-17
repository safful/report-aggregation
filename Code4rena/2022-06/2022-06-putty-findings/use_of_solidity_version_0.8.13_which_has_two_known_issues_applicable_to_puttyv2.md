## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed
- old-submission-method

# [Use of Solidity version 0.8.13 which has two known issues applicable to PuttyV2](https://github.com/code-423n4/2022-06-putty-findings/issues/348) 

# Lines of code

https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L2


# Vulnerability details

The solidity version 0.8.13 has below two issues applicable to PuttyV2
1) Vulnerability related to ABI-encoding.
ref : https://blog.soliditylang.org/2022/05/18/solidity-0.8.14-release-announcement/
This vulnerability can be misused since the function hashOrder() and hashOppositeOrder() has applicable conditions.
"...pass a nested array directly to another external function call or use abi.encode on it."

2) Vulnerability related to 'Optimizer Bug Regarding Memory Side Effects of Inline Assembly'
ref : https://blog.soliditylang.org/2022/06/15/solidity-0.8.15-release-announcement/
PuttyV2 inherits solidity contracts from openzeppelin and solmate, and both these uses inline assembly, and optimization is enabled while compiling.

## Recommended Mitigation Steps
Use recent Solidity version 0.8.15 which has the fix for these issues



