## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Reducing the epoch length results in leaking value from advancement incentives](https://github.com/code-423n4/2021-11-malt-findings/issues/4) 

# Handle

TomFrench


# Vulnerability details

## Impact

Unintended advancement incentives being paid out to third party

## Proof of Concept

`DAO.sol` incentives outside parties to advance the epoch by minting 100 MALT tokens for calling the `advance` function. This is limited by checking that the start timestamp of the next epoch has passed.
 
https://github.com/code-423n4/2021-11-malt/blob/d3f6a57ba6694b47389b16d9d0a36a956c5e6a94/src/contracts/DAO.sol#L55-L63

This start timestamp is calculated by multiplying the new epoch number by the length of an epoch and adding it to the genesis timestamp.

https://github.com/code-423n4/2021-11-malt/blob/d3f6a57ba6694b47389b16d9d0a36a956c5e6a94/src/contracts/DAO.sol#L65-L67

This method makes no accommodation for the fact that previous epochs may have been set to be a different length to what they are currently.

https://github.com/code-423n4/2021-11-malt/blob/d3f6a57ba6694b47389b16d9d0a36a956c5e6a94/src/contracts/DAO.sol#L111-L114

In the case where the epoch length is reduced, `DAO` will think that the epoch number can be incremented potentially many times. Provided the `advanceIncentive` is worth more than the gas necessary to advance the epoch will be rapidly advanced potentially many times paying out unnecessary incentives.

## Recommended Mitigation Steps

Rather than calculating from the genesis timestamp, store the last time that the epoch length was modified and calculate from there.

