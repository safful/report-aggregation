## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- resolved

# [Usage of deprecated `safeApprove`](https://github.com/code-423n4/2021-10-pooltogether-findings/issues/18) 

# Handle

cmichel


# Vulnerability details

Description: `safeApprove` is now deprecated, see [this comment](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/566a774222707e424896c0c390a84dc3c13bdcb2/contracts/token/ERC20/utils/SafeERC20.sol#L38).

## Impact
When using one of these unsupported tokens, all transactions revert and the protocol cannot be used.

## Recommended Mitigation Steps
As per OpenZepplin documentation “whenever possible, use `safeIncreaseAllowance` and `safeDecreaseAllowance` instead”.


