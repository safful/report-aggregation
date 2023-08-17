## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed
- reviewed

# [User can steal all rewards due to checkpoint after transfer](https://github.com/code-423n4/2022-04-backd-findings/issues/36) 

# Lines of code

https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/contracts/StakerVault.sol#L112-L119


# Vulnerability details

## Impact
I believe this to be a high severity vulnerability that is potentially included in the currently deployed `StakerVault.sol` contract also. The team will be contacted immediately following the submission of this report.

In `StakerVault.sol`, the user checkpoints occur AFTER the balances are updated in the `transfer()` function. The user checkpoints update the amount of rewards claimable by the user. Since their rewards will be updated after transfer, a user can send funds between their own accounts and repeatedly claim maximum rewards since the pool's inception.

In every actionable function except `transfer()` of `StakerVault.sol`, a call to `ILpGauge(lpGauge).userCheckpoint()` is correctly made BEFORE the action effects.

## Proof of Concept
Assume a certain period of time has passed since the pool's inception. For easy accounting, assume `poolStakedIntegral` of `LpGauge.sol` equals `1`. The `poolStakedIntegral` is used to keep track of the current reward rate.

Steps:
- Account A stakes 1000 LP tokens. `balances[A] += 1000` 
- In the same `stakeFor()` function, `userCheckpoint()` was already called so A will already have `perUserShare[A]` set correctly based on their previously 0 balance and the current `poolStakedIntegral`.
- Account A can immediately send all balance to Account B via `transfer()`.
- Since the checkpoint occurs after the transfer, B's balance will increase and then `perUserShare[B]` will be updated. The calculation for `perUserShare` looks as follows.

```
perUserShare[user] += (
            (stakerVault.stakedAndActionLockedBalanceOf(user)).scaledMul(
                (poolStakedIntegral_ - perUserStakedIntegral[user])
            )
        );
```

Assuming Account B is new to the protocol, their `perUserStakedIntegral[user]` will default to `0`.

`perUserShare[B] += 1000 * (1 - 0) = 1000`

- B is able to call `claimRewards()` and mint all 1000 reward tokens.
- B then calls `transfer()` and sends all 1000 staked tokens to Account C.
- Same calculation occurs, and C can claim all 1000 reward tokens.
- This process can be repeated until the contract is drained of reward tokens.

## Tools Used
Static review.

## Recommended Mitigation Steps
In `StakerVault.transfer()`, move the call to `ILpGauge(lpGauge).userCheckpoint()` to before the balances are updated.

