## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Lack of maximum and minimum vesting amount check on the vesting function](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/32) 

# Handle

defsec


# Vulnerability details

## Impact

During the code review, It has been seen maxVesting amount is disabled. However, there is no maximum and minimum vesting amount defined. Users can vest small amount. For the protocol liquditiy calculation maximum and minimum threshold should be defined. 

## Proof of Concept

1. Navigate to the following contract.

"""
https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/Vesting.sol#L76
"""

2. Vesting amount didnt check.

## Tools Used

Review

## Recommended Mitigation Steps

It is suggested to check maximum/minimum vesting amount on the contract. 

