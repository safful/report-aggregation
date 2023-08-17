## Tags

- bug
- 3 (High Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- H-03

# [Theft of ETH of free floating SLOT holders](https://github.com/code-423n4/2022-11-stakehouse-findings/issues/40) 

# Lines of code

https://github.com/code-423n4/2022-11-stakehouse/blob/39a3a84615725b7b2ce296861352117793e4c853/contracts/syndicate/Syndicate.sol#L369
https://github.com/code-423n4/2022-11-stakehouse/blob/39a3a84615725b7b2ce296861352117793e4c853/contracts/syndicate/Syndicate.sol#L668
https://github.com/code-423n4/2022-11-stakehouse/blob/39a3a84615725b7b2ce296861352117793e4c853/contracts/syndicate/Syndicate.sol#L228


# Vulnerability details

## Impact

A malicious user can steal all claimable ETH belonging to free floating SLOT holders...

## Proof of Concept

https://gist.github.com/clems4ever/f1149743897b2620eab0734f88208603

run it in the test suite with forge

## Tools Used

Manual review / forge

## Recommended Mitigation Steps

+= operator instead of =    in https://github.com/code-423n4/2022-11-stakehouse/blob/39a3a84615725b7b2ce296861352117793e4c853/contracts/syndicate/Syndicate.sol#L228 ?

The logic for keeping the rewards up-to-date is also quite complex in my opinion. The main thing that triggered it for me was the lazy call to `updateAccruedETHPerShares`. Why not keeping the state updated after each operation instead?