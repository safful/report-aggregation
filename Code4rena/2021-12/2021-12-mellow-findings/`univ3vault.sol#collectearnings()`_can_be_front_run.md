## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed

# [`UniV3Vault.sol#collectEarnings()` can be front run](https://github.com/code-423n4/2021-12-mellow-findings/issues/98) 

# Handle

WatchPug


# Vulnerability details

For `UniV3Vault`, it seems that lp fees are collected through `collectEarnings()` callable by the `strategy` and reinvested (rebalanced).

However, in the current implementation, unharvested yields are not included in `tvl()`, making it vulnerable to front-run attacks that steal pending yields.

https://github.com/code-423n4/2021-12-mellow/blob/6679e2dd118b33481ee81ad013ece4ea723327b5/mellow-vaults/contracts/UniV3Vault.sol#L100-L122

https://github.com/code-423n4/2021-12-mellow/blob/6679e2dd118b33481ee81ad013ece4ea723327b5/mellow-vaults/contracts/UniV3Vault.sol#L80-L97

### POC

Given:

- Current `tvl()` is `10 ETH` and `40,000 USDC`;
- Current unclaimed yields (trading fees) is `1 ETH` and `4,000 USDC`;

1. `strategy` calls `collectEarnings()` to collect fees and reinvest;
2. The attacker sends a deposit tx with a higher gas price to deposit `10 ETH` and `40,000 USDC`, take 50% share of the pool;
3. After the transaction in step 1 is packed, the attacker calls `withdraw()` and retrieves `10.5 ETH` and `42,000 USDC`.

As a result, the attacker has stolen half of the pending yields in about 1 block of time.

### Recommendation

Consider including fees in `tvl()`.

For the code to calculate fees earned, please reference `_computeFeesEarned()` in G-UNI project:

https://github.com/gelatodigital/g-uni-v1-core/blob/master/contracts/GUniPool.sol#L762-L806

