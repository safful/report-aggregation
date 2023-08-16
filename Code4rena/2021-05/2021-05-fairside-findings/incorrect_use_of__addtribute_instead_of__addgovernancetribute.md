## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- resolved

# [Incorrect use of _addTribute instead of _addGovernanceTribute](https://github.com/code-423n4/2021-05-fairside-findings/issues/20) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The addRegistrationTributeGovernance() function is called by the FSD network to update tribute when 7.5% is contributed towards governance as part of purchaseMembership(). However, this function incorrectly calls _addTribute() (as done in addRegistrationTribute) instead of _addGovernanceTribute().

The impact is that governanceTributes never gets updated and the entire tribute accounting logic is rendered incorrect.


## Proof of Concept

https://github.com/code-423n4/2021-05-FairSide/blob/3e9f6d40f70feb67743bdc70d7db9f5e3a1c3c96/contracts/token/FSD.sol#L140

https://github.com/code-423n4/2021-05-FairSide/blob/3e9f6d40f70feb67743bdc70d7db9f5e3a1c3c96/contracts/token/FSD.sol#L130

https://github.com/code-423n4/2021-05-FairSide/blob/3e9f6d40f70feb67743bdc70d7db9f5e3a1c3c96/contracts/network/FSDNetwork.sol#L195

https://github.com/code-423n4/2021-05-FairSide/blob/3e9f6d40f70feb67743bdc70d7db9f5e3a1c3c96/contracts/dependencies/TributeAccrual.sol#L30-L48

https://github.com/code-423n4/2021-05-FairSide/blob/3e9f6d40f70feb67743bdc70d7db9f5e3a1c3c96/contracts/dependencies/TributeAccrual.sol#L50-L70


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Use _addGovernanceTribute() instead of _addTribute on L140 of FSD.sol

