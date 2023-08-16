## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Usage of deprecated safeApprove](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/166) 

# Handle

mics


# Vulnerability details

safeApprove is now deprecated, see the link below.
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/566a774222707e424896c0c390a84dc3c13bdcb2/contracts/token/ERC20/utils/SafeERC20.sol#L38.

This appears for example in line 499 of AirdropDistribution.sol.

we recommend as in OpenZepplin documentation “whenever possible, use safeIncreaseAllowance and safeDecreaseAllowance instead”.




