## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- Resolved

# [add zero address validation in constructor](https://github.com/code-423n4/2021-08-realitycards-findings/issues/61) 

# Handle

JMukesh


# Vulnerability details

## Impact
since the parameter in the constructor are used to initialize the state variable , proper check up should be done , other wise error in these state variable  can lead to redeployment of contract

## Proof of Concept

https://github.com/code-423n4/2021-08-realitycards/blob/39d711fdd762c32378abf50dc56ec51a21592917/contracts/RCLeaderboard.sol#L50

https://github.com/code-423n4/2021-08-realitycards/blob/39d711fdd762c32378abf50dc56ec51a21592917/contracts/RCOrderbook.sol#L136

https://github.com/code-423n4/2021-08-realitycards/blob/39d711fdd762c32378abf50dc56ec51a21592917/contracts/RCTreasury.sol#L120

## Tools Used
manual review

## Recommended Mitigation Steps
add zero address validation

