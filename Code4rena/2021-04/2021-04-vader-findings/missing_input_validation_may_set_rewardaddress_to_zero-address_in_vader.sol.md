## Tags

- bug
- disagree with severity
- 1 (Low Risk)
- sponsor confirmed

# [Missing input validation may set rewardAddress to zero-address in Vader.sol](https://github.com/code-423n4/2021-04-vader-findings/issues/160) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Function setRewardAddress is used by DAO to change rewardAddress from USDV to something else. However, there is no zero-address validation on the address. This may accidentally mint rewards to zero-address.

## Proof of Concept

https://github.com/code-423n4/2021-04-vader/blob/3041f20c920821b89d01f652867d5207d18c8703/vader-protocol/contracts/Vader.sol#L80

https://github.com/code-423n4/2021-04-vader/blob/3041f20c920821b89d01f652867d5207d18c8703/vader-protocol/contracts/Vader.sol#L209

https://github.com/code-423n4/2021-04-vader/blob/3041f20c920821b89d01f652867d5207d18c8703/vader-protocol/contracts/Vader.sol#L183-L186


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Add zero-address check to setRewardAddress.

