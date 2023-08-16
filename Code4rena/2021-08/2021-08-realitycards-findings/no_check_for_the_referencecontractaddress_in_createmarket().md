## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- Resolved

# [No check for the referenceContractAddress in createMarket()](https://github.com/code-423n4/2021-08-realitycards-findings/issues/50) 

# Handle

JMukesh


# Vulnerability details

## Impact

 referenceContractAddress  is used in createMarket() to create newAddress  for the market , a necessary check should be there that referenceContractAddress exist or not, because if createMarket() is called before setReferenceContractAddress() address(0) will be passed as referenceContractAddress , since addMarket() of treasury and nfthub does not have address validation for the market

## Proof of Concept
https://github.com/code-423n4/2021-08-realitycards/blob/39d711fdd762c32378abf50dc56ec51a21592917/contracts/RCFactory.sol#L714

## Tools Used

manual review

## Recommended Mitigation Steps

add a condition to check the referenceContractAddress

