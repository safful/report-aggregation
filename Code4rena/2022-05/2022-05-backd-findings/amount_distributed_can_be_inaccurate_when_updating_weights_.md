## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [Amount distributed can be inaccurate when updating weights ](https://github.com/code-423n4/2022-05-backd-findings/issues/47) 

# Lines of code

 https://github.com/code-423n4/2022-05-backd/blob/2a5664d35cde5b036074edef3c1369b984d10010/protocol/contracts/tokenomics/Minter.sol#L220
 https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/contracts/tokenomics/InflationManager.sol#L559
 https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/contracts/tokenomics/InflationManager.sol#L572
 https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/contracts/tokenomics/InflationManager.sol#L586


# Vulnerability details


## Impact

When updating pool inflation rates, other pools see their `currentRate` being modified without having `poolCheckpoint` called, which leads to false computations.

This will lead to either users losing a part of their claims, but can also lead to too many tokens could be distributed, preventing some users from claiming due to the `totalAvailableToNow` requirement in `Minter`.

## Proof of concept

Imagine you have 2 AMM pools A and B, both with an `ammPoolWeight` of 100, where `poolCheckpoint` has not been called for a moment. Then, imagine calling [`executeAmmTokenWeight`](https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/contracts/tokenomics/InflationManager.sol#L318) to reduce the weight of A to 0. 

Only A is checkpointed [here](https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/contracts/tokenomics/InflationManager.sol#L591), so when B will be checkpointed it will call `getAmmRateForToken`, which will see a pool weight of 100 and a total weight of 100 over the whole period since the last checkpoint of B, which is false, therefore it will distribute too many tokens. This is critical has the minter expects an exact or lower than expected distribution due to the requirement of `totalAvailableToNow`.

In the opposite direction, when increasing weights, it will lead to less tokens being distributed in some pools than planned, leading to a loss for users.

## Mitigation steps
Checkpoint every `LpStakerVault`, `KeeperGauge` or `AmmGauge` when updating the weights of one of them.



