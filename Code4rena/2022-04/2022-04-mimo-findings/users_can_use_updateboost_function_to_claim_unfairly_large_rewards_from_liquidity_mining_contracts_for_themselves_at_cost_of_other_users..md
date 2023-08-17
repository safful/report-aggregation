## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Users can use updateBoost function to claim unfairly large rewards from liquidity mining contracts for themselves at cost of other users.](https://github.com/code-423n4/2022-04-mimo-findings/issues/136) 

# Lines of code

https://github.com/code-423n4/2022-04-mimo/blob/b18670f44d595483df2c0f76d1c57a7bfbfbc083/core/contracts/liquidityMining/v2/PARMinerV2.sol#L159-L165
https://github.com/code-423n4/2022-04-mimo/blob/b18670f44d595483df2c0f76d1c57a7bfbfbc083/core/contracts/liquidityMining/v2/GenericMinerV2.sol#L88-L94


# Vulnerability details

## Impact
Users aware of this vulnerability could effectively steal a portion of liquidity mining rewards from honest users.

Affected contracts are: `SupplyMinerV2`, ` DemandMinerV2`, ` PARMinerV2` 

`VotingMinerV2` is less affected because locking veMIMO in `votingEscrow`  triggers a call to `releaseMIMO` of this miner contract (which in turn updates user's boost multiplier). 

## Proof of Concept
Let's focus here on `SupplyMinerV2`. The exploits for other liquidity mining contracts are analogous.

### Scenario 1:

Both Alice and Bob deposit 1 WETH to `coreVaults`  and borrow 100 PAR from `coreVaults`. They both have no locked veMIMO. 

Now they wait for a month without interacting with the protocol. In the meantime, `SupplyMinerV2` accumulated 100 MIMO for rewards.

Alice locks huge amount of veMIMO in `votingEscrow`, so now her `boostMultiplier`  is 4.

Let's assume that Alice and Bob are the only users of the protocol. Because they borrowed the same amounts of PAR, they should have the same stakes for past month, so a fair reward for each of them (for this past month) should be 50 MIMO. If they simply repay their debts now, 50 MIMO is indeed what they get.

However if Alice calls `supplyMiner.updateBoost(alice)` before repaying her debt, she can claim 80 MIMO and leave only 20 MIMO for Bob. She can basically apply the multiplier 4 to her past stake.

### Scenario 2:

Both Alice and Bob deposit 1 WETH to `coreVaults`  and borrow 100 PAR from `coreVaults`. Bob locks huge amount of veMIMO in `votingEscrow` for 4 years, so now his `boostMultiplier` is 4. 

Alice and Bob wait for 4 years without interacting with the protocol.

`SupplyMinerV2` accumulated 1000 MIMO rewards. 

Because of his locked veMIMO, Bob should be able to claim larger reward than Alice. Maybe not 4 times larger but definitely larger.

However, if Alice includes  a transaction with call `supplyMiner.updateBoost(bob)` before Bob's `vaultsCore.repay()` , then she can claim 500 MIMO. She can effectively set Bob's `boostMultiplier` for past 4 years to 1.

## Tools Used
Tested in Foundry

## Recommended Mitigation Steps
I have 2 ideas:
1. Remove `updateBoost` function. There shouldn't be a way to update boost multiplier without claiming rewards and updating `_userInfo.accAmountPerShare` .  So `releaseRewards`  should be sufficient.
2. A better, but also much more difficult solution, would be to redesign boost updates in such a way that distribution of rewards no longer depends on when and how often boost multiplier is updated. If the formula for boost multiplier stays the same, this approach might require calculating integrals of the multiplier as a function of time.

