## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [MerkleResistor: zero coinsPerSecond will brick tranche initialization and withdrawals](https://github.com/code-423n4/2022-05-factorydao-findings/issues/107) 

# Lines of code

https://github.com/code-423n4/2022-05-factorydao/blob/db415804c06143d8af6880bc4cda7222e5463c0e/contracts/MerkleResistor.sol#L259
https://github.com/code-423n4/2022-05-factorydao/blob/db415804c06143d8af6880bc4cda7222e5463c0e/contracts/MerkleResistor.sol#L264


# Vulnerability details

## Details & Impact

It is possible for `coinsPerSecond` to be zero. In these cases, the `startTime` calculation 

```solidity
uint startTime = block.timestamp + vestingTime - (totalCoins / coinsPerSecond);
```

will revert from division by zero, preventing initialization, and by extension, withdrawals of vested tokens.

## Proof of Concept

We assume vesting time chosen is the maximum (`tree.maxEndTime`) so that `totalCoins = maxTotalPayments`. These examples showcase some possibilities for which the calculated `coinsPerSecond` can be zero.

### Example 1: High upfront percentage

- `pctUpFront = 99` (99% up front)
- `totalCoins = 10_000e6` (10k USDC)
- `vestingTime = 1 year`

```solidity
uint coinsPerSecond = (totalCoins * (uint(100) - tree.pctUpFront)) / (vestingTime * 100);
// 10_000e6 * (100 - 99) / (365 * 86400 * 100)
// = 0
```

### Example 2: Small reward amount / token decimals

- `pctUpFront = 0`
- `totalCoins = 100_000e2` (100k EURS)
- `vestingTime = 180 days`

```solidity
uint coinsPerSecond = (totalCoins * (uint(100) - tree.pctUpFront)) / (vestingTime * 100);
// 100_000e2 * 100 / (180 * 86400 * 100)
// = 0
```

## Recommended Mitigation Steps

Scale up `coinsPerSecond` by `PRECISION`, then scale down when executing withdrawals. While it isn’t foolproof, the possibility of `coinsPerSecond` being zero is reduced significantly.

```solidity
// L264
uint coinsPerSecond = (totalCoins * (uint(100) - tree.pctUpFront)) * PRECISION / (vestingTime * 100);

// L184
currentWithdrawal = (block.timestamp - tranche.lastWithdrawalTime) * tranche.coinsPerSecond / PRECISION;
```

