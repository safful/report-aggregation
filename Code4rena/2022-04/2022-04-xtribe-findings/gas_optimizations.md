## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas Optimizations](https://github.com/code-423n4/2022-04-xtribe-findings/issues/84) 

1. 
Title: Using delete statement to empty `rewardsAccrued` can save gas

Proof of Concept:
https://github.com/fei-protocol/flywheel-v2/blob/77bfadf388db25cf5917d39cd9c0ad920f404aad/src/FlywheelCore.sol#L123

Recommended Mitigation Steps:
Change to:
```
	delete rewardsAccrued[user];
```

========================================================================

2. 
Title: Using != is more gas efficient

Proof of Concept:
https://github.com/fei-protocol/flywheel-v2/blob/77bfadf388db25cf5917d39cd9c0ad920f404aad/src/FlywheelCore.sol#L167
https://github.com/fei-protocol/flywheel-v2/blob/77bfadf388db25cf5917d39cd9c0ad920f404aad/src/FlywheelCore.sol#L218

Recommended Mitigation Steps:
Change to:
```
	if (oldRewardBalance != 0) {
```

========================================================================

3.
Title: Using `storage` to declare Struct variable inside function

Proof of Concept:
https://github.com/fei-protocol/flywheel-v2/blob/77bfadf388db25cf5917d39cd9c0ad920f404aad/src/FlywheelCore.sol#L85
https://github.com/fei-protocol/flywheel-v2/blob/77bfadf388db25cf5917d39cd9c0ad920f404aad/src/FlywheelCore.sol#L106

Recommended Mitigation Steps:
instead of caching `RewardsState` to memory. read it directly from storage.
```
	RewardsState storage state = strategyState[strategy];
```

========================================================================

4.
Title: Using `calldata` on struct parameter

Proof of Concept:
https://github.com/fei-protocol/flywheel-v2/blob/77bfadf388db25cf5917d39cd9c0ad920f404aad/src/FlywheelCore.sol#L210
https://github.com/fei-protocol/flywheel-v2/blob/77bfadf388db25cf5917d39cd9c0ad920f404aad/src/FlywheelCore.sol#L241

Recommended Mitigation Steps:
Using `calldata` to store struct data type can save gas
```
	function accrueStrategy(ERC20 strategy, RewardsState calldata state)
```

========================================================================

5.
Title: Using unchecked to calculate can save gas

Proof of Concept:
https://github.com/fei-protocol/flywheel-v2/blob/77bfadf388db25cf5917d39cd9c0ad920f404aad/src/rewards/FlywheelStaticRewards.sol#L60

Recommended Mitigation Steps:
`rewards.rewardsEndTimestamp` is checked that it won't `>` than `lastUpdatedTimestamp`
```
unchecked{
	elapsed = rewards.rewardsEndTimestamp - lastUpdatedTimestamp;
}
```

========================================================================

6.
Title: Using > is cheaper than >=

Proof of Concept:
https://github.com/Rari-Capital/solmate/blob/9f16db2144cc9a7e2ffc5588d4bf0b66784283bd/src/tokens/ERC20.sol#L125

Recommended Mitigation Steps:
just use `>` can save gas
Change to:
```
	require(deadline > block.timestamp, "PERMIT_DEADLINE_EXPIRED");
```

========================================================================

7.
Title: Using `immutable` can save gas

Proof of Concept:
https://github.com/Rari-Capital/solmate/blob/9f16db2144cc9a7e2ffc5588d4bf0b66784283bd/src/tokens/ERC20.sol#L23

Recommended Mitigation Steps:
use `immutable` to declare variable which set once in constructor

========================================================================

8.
Title: Using multiple `require` instead `&&` can save gas

Proof of Concept:
https://github.com/Rari-Capital/solmate/blob/9f16db2144cc9a7e2ffc5588d4bf0b66784283bd/src/tokens/ERC20.sol#L154

Recommended Mitigation Steps:
Change to:
```
	require(recoveredAddress != address(0), "INVALID_SIGNER");
	require(recoveredAddress == owner, "INVALID_SIGNER");
```

========================================================================

9.
Title: unnecessary value set. the default value of uint is 0.

Proof of Concept:
https://github.com/fei-protocol/flywheel-v2/blob/77bfadf388db25cf5917d39cd9c0ad920f404aad/src/token/ERC20Gauges.sol#L134
https://github.com/fei-protocol/flywheel-v2/blob/77bfadf388db25cf5917d39cd9c0ad920f404aad/src/token/ERC20Gauges.sol#L184
https://github.com/fei-protocol/flywheel-v2/blob/77bfadf388db25cf5917d39cd9c0ad920f404aad/src/token/ERC20Gauges.sol#L307
https://github.com/fei-protocol/flywheel-v2/blob/77bfadf388db25cf5917d39cd9c0ad920f404aad/src/token/ERC20Gauges.sol#L384

Recommended Mitigation Steps:
remove 0 value can save gas

========================================================================