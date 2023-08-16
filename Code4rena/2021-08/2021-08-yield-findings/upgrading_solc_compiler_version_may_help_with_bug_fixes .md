## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Upgrading solc compiler version may help with bug fixes ](https://github.com/code-423n4/2021-08-yield-findings/issues/50) 

# Handle

0xRajeev


# Vulnerability details

## Impact

solc version 0.8.3 and 0.8.4 fixed important bugs in the compiler. Using version 0.8.1 misses these fixes and may cause a vulnerability.

## Proof of Concept

https://github.com/code-423n4/2021-08-yield/blob/4dc46470e616dd0cbd9db9b4742e36c4d809e02c/contracts/utils/token/ERC20Rewards.sol#L2

https://github.com/ethereum/solidity/releases/tag/v0.8.4: Solidity 0.8.4 fixes a bug in the ABI decoder. The release contains an important bugfix. See decoding from memory bug blog post for more details.

https://github.com/ethereum/solidity/releases/tag/v0.8.3: Solidity 0.8.3 is a bugfix release that fixes an important bug about how the optimizer handles the Keccak256 opcode.
For details on the bug, please see the bug blog post.


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Consider upgrading to 0.8.3 or 0.8.4

