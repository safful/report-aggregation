## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Redundant check on claim](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/97) 

# Handle

fr0zn


# Vulnerability details

## Vulnerability Details
On the `claim` function ([line 524](https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/AirdropDistribution.sol#L524))  of the `AirdropDistribution.sol` contract, the `validated` check is redundant. The flag is only set when the `validate` function is called. Once validated, the amount will always be different than zero, meaning that the check is not necessary.

## Impact
Gas optimization

## Tools Used
Manual code review

## Recommended Mitigation Steps
The check could be removed for gas optimization.

