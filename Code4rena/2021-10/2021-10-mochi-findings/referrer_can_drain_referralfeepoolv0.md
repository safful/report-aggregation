## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Referrer can drain ReferralFeePoolV0](https://github.com/code-423n4/2021-10-mochi-findings/issues/55) 

# Handle

gzeon


# Vulnerability details

## Impact
function claimRewardAsMochi in ReferralFeePoolV0.sol did not reduce user reward balance, allowing referrer to claim the same reward repeatedly and thus draining the fee pool.

## Proof of Concept
https://github.com/code-423n4/2021-10-mochi/blob/main/projects/mochi-core/contracts/feePool/ReferralFeePoolV0.sol
L28-47 did not reduce user reward balance

## Tools Used
None

## Recommended Mitigation Steps
Add the following lines
> rewards -= reward[msg.sender];
> reward[msg.sender] = 0;


