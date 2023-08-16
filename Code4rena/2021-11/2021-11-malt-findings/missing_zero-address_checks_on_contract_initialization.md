## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Missing zero-address checks on contract initialization](https://github.com/code-423n4/2021-11-malt-findings/issues/74) 

# Handle

hyh


# Vulnerability details

## Impact

Being instantiated with wrong configuration, the contract is inoperable and deploy gas costs will be lost.
If misconfiguration is noticed too late the various types of malfunctions become possible.

## Proof of Concept

The checks for zero addresses during contract construction and initialization are considered to be the best-practice.

Now basically all the contract do not check for correctness of constructor arguments:

https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/Malt.sol#L29

https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/RewardSystem/RewardOverflowPool.sol#L25

https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/TransferService.sol#L25

https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/ForfeitHandler.sol#L31

https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/MiningService.sol#L30

https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/DexHandlers/UniswapHandler.sol#L47

...

## Recommended Mitigation Steps

Add zero-address checks and key non-address variables checks in all contract constructors. Small increase of gas costs are far out weighted by wrong deploy costs savings and additional coverage against misconfiguration.


