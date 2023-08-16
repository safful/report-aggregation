## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- disagree with severity

# [Flash loan manipulation on `getPoolShareWeight` of `Utils`](https://github.com/code-423n4/2021-07-spartan-findings/issues/238) 

# Handle

shw


# Vulnerability details

## Impact

The `getPoolShareWeight` function returns a user's pool share weight by calculating how many SPARTAN the user's LP tokens account for. However, this approach is vulnerable to flash loan manipulation since an attacker can swap a large number of TOKEN to SPARTAN to increase the number of SPARTAN in the pool, thus effectively increasing his pool share weight.

## Proof of Concept

According to the implementation of `getPoolShareWeight,` a user's pool share weight is calculated by `uints * baseAmount / totalSupply`, where `uints` is the number of user's LP tokens, `totalSupply` is the total supply of LP tokens, and `baseAmount` is the number of SPARTAN in the pool. Thus, a user's pool share weight is proportional to the number of SPARTAN in the pool. Consider the following attack scenario:

1. Supposing the attacked pool is SPARTAN-WBNB. The attacker first prepares some LP tokens (WBNB-SPP) by adding liquidity to the pool.
2. The attacker then swaps a large number of WBNB to SPARTAN, which increases the pool's `baseAmount`. He could split his trade into small amounts to reduce slip-based fees.
3. The attacker now wants to increase his weight in the `DaoVault`. He adds his LP tokens to the pool by calling the `deposit` function of `Dao.`
4. `Dao` then calls `depositLP` of `DaoVault`, causing the attacker's weight to be recalculated. Due to the large proportion of SPARTAN in the pool, the attacker's weight is artificially increased.
5. With a higher member weight, the attacker can, for example, vote the current proposal with more votes than he should have or obtain more rewards when calling `harvest` of the `Dao` contract.
6. The attacker then swaps back SPARTAN to WBNB and only loses the slip-based fees.

Referenced code:
[Utils.sol#L46-L50](https://github.com/code-423n4/2021-07-spartan/blob/main/contracts/Utils.sol#L46-L50)
[Utils.sol#L70-L77](https://github.com/code-423n4/2021-07-spartan/blob/main/contracts/Utils.sol#L70-L77)
[DaoVault.sol#L44-L56](https://github.com/code-423n4/2021-07-spartan/blob/main/contracts/DaoVault.sol#L44-L56)
[Dao.sol#L201](https://github.com/code-423n4/2021-07-spartan/blob/main/contracts/Dao.sol#L201)
[Dao.sol#L570](https://github.com/code-423n4/2021-07-spartan/blob/main/contracts/Dao.sol#L570)

## Recommended Mitigation Steps

A possible mitigation is to record the current timestamp when a user's weight in the `DaoVault` or `BondVault` is recalculated and force the new weight to take effect only after a certain period, e.g., a block time. This would prevent the attacker from launching the attack since there is typically no guarantee that he could arbitrage the WBNB back in the next block.

