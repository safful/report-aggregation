## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Use immutable keyword](https://github.com/code-423n4/2021-07-wildcredit-findings/issues/28) 

# Handle

gpersoon


# Vulnerability details

## Impact
Some of the constructors set values that are never changed. See proof of concept.
Its best to use the immutable keyword to make sure they aren't changed by accident.

## Proof of Concept
//https://github.com/code-423n4/2021-07-wildcredit/blob/main/contracts/LendingPair.sol#L38
address public tokenA;
address public tokenB;

//https://github.com/code-423n4/2021-07-wildcredit/blob/main/contracts/RewardDistribution.sol#L37
IPairFactory public factory;
IController  public controller;
IERC20  public rewardToken;

## Tools Used

## Recommended Mitigation Steps
Add the immutable keyword where possible


