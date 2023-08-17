## Tags

- bug
- 3 (High Risk)
- sponsor acknowledged
- sponsor confirmed

# [Frontrunning by malicious validator](https://github.com/code-423n4/2022-09-frax-findings/issues/81) 

# Lines of code

https://github.com/code-423n4/2022-09-frax/blob/main/src/frxETHMinter.sol#L120


# Vulnerability details

## Impact
Frontrunning by malicious validator changing withdrawal credentials

## Proof of Concept
A malicious validator can frontrun depositEther transaction for its pubKey and deposit 1 ether for different withdrawal credential, thereby setting withdrawal credit before deposit of 32 ether by contract and thereby when 32 deposit ether are deposited, the withdrawal credential is also what was set before rather than the one being sent in depositEther transaction

## Recommended Mitigation Steps
Set withdrawal credentials for validator by depositing 1 ether with desired withdrawal credentials, before adding it in Operator Registry