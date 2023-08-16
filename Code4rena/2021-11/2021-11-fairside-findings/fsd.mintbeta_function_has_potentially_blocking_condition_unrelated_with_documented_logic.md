## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [FSD.mintBeta function has potentially blocking condition unrelated with documented logic](https://github.com/code-423n4/2021-11-fairside-findings/issues/96) 

# Handle

hyh


# Vulnerability details

## Impact

As funding pool should be filled with Hatch phase deposits, the phase advance should happen only after it is filled, but when this happen the minting in Beta phase would be frozen by 'fundingPool.balance < 2000 ether' condition. As mintBeta is the only logic for Beta phase the contract will be frozen until phase change.

If there is a setup when Hatch phase advances to Beta before funding pool is filled, mintBeta will work only while it has below 2000 ether, i.e. mintBeta behavior will not be controlled explicitly: anyone can end the Beta phase by sending enough ether directly to funding pool and the contract mint will be frozen until next phase advance. 

## Proof of Concept

'fundingPool.balance < 2000 ether' condition for minting in Beta phase can be blocking as funding pool transfers happen only in mintHatch function during Hatch phase, while subsequent phases do not have any funding pool related logic neither in code, nor in documentation.
https://github.com/code-423n4/2021-11-fairside/blob/main/contracts/token/FSD.sol#L204
https://fairside-network.gitbook.io/fairside-network/white-paper/augmented-bonding-curve

## Recommended Mitigation Steps

Remove 'fundingPool.balance < 2000 ether' condition from mintBeta

