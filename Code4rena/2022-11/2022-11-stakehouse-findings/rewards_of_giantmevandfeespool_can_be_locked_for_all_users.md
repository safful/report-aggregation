## Tags

- bug
- 3 (High Risk)
- primary issue
- selected for report
- sponsor confirmed
- H-02

# [Rewards of GiantMevAndFeesPool can be locked for all users](https://github.com/code-423n4/2022-11-stakehouse-findings/issues/33) 

# Lines of code

https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantMevAndFeesPool.sol#L172
https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantLP.sol#L8


# Vulnerability details

## Impact
Any malicious user could make the rewards in GiantMevAndFeesPool inaccessible to all other users...

## Proof of Concept

https://gist.github.com/clems4ever/9b05391cc2192c1b6e8178faa38dfe41

Copy the file in the test suite and run the test.

## Tools Used

forge test

## Recommended Mitigation Steps

Protect the inherited functions of the ERC20 tokens (GiantLP and LPToken) because `transfer` is not protected and can trigger the `before` and `after` hooks. There is the same issue with LPToken and StakingFundsVault.