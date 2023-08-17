## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [Guaranteed citadel profit](https://github.com/code-423n4/2022-04-badger-citadel-findings/issues/71) 

# Lines of code

https://github.com/code-423n4/2022-04-badger-citadel/blob/main/src/CitadelMinter.sol#L217


# Vulnerability details

## Impact
User can sandwich `mintAndDistribute` function if mintable is high enough
- Deposit before
- Withdraw after
- Take after 21 days citadels

## Proof of Concept
`mintAndDistribute` increase a price of staking share, that allows to withdraw more than deposited.
user takes part of distributed citadels, so different users have smaller profit from distribution

## Tools Used

## Recommended Mitigation Steps
Call `mintAndDistribute` through flashbots

