## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- resolved

# [Lack of zero address validation ](https://github.com/code-423n4/2021-06-realitycards-findings/issues/56) 

# Handle

JMukesh


# Vulnerability details

## Impact
  constructor of RCorderbook.sol lacks zero address validation , since parameter of costructor are used initialize state variable which are used in other function of the contract , error in these state variable can lead to redeployment of contract

## Proof of Concept

https://github.com/code-423n4/2021-06-realitycards/blob/main/contracts/RCOrderbook.sol#L106

## Tools Used
manual review

## Recommended Mitigation Steps
add require condition to check for zero address

