## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed

# [LP pricing formula is vulnerable to flashloan manipulation](https://github.com/code-423n4/2022-01-behodler-findings/issues/304) 

# Handle

shw


# Vulnerability details

## Impact

The LP pricing formula used in the `burnAsset` function of `LimboDAO` is vulnerable to flashloan manipulation. By swapping a large number of EYE into the underlying pool, an attacker can intentionally inflate the value of the LP tokens to get more `fate` than he is supposed to with a relatively low cost.

With the large portion of `fate` he gets, he has more voting power to influence the system's decisions, or even he can convert his `fate` to Flan tokens for a direct profit.

## Proof of Concept

Below is an example of how the attack works:

1. Suppose that there are 1000 EYE and 1000 LINK tokens in the UniswapV2 LINK-EYE pool. The pool's total supply is 1000, and the attacker has 100 LP tokens.
2. If the attacker burns his LP tokens, he earns `1000 * 100/1000 * 20 = 2000` amount of `fate`.
3. Instead, the attacker swaps in 1000 EYE and gets 500 LINK from the pool (according to `x * y = k`, ignoring fees for simplicity). Now the pool contains 2000 EYE and 500 LINK tokens.
4. After the manipulation, he burns his LP tokens and gets `2000 * 100/1000 * 20 = 4000` amount of `fate`.
5. Lastly, he swaps 500 LINK into the pool to get back his 1000 EYE.
6. Compared to Step 2, the attacker earns a double amount of `fate` by only paying the swapping fees to the pool. The more EYE tokens he swaps into the pool, the more `fate` he can get. This attack is practically possible by leveraging flashloans or flashswaps from other pools containing EYE tokens.

The `setEYEBasedAssetStake` function has the same issue of using a manipulatable LP pricing formula. For more detailed explanations, please refer to the analysis of the [Cheese Bank attack](https://peckshield.medium.com/cheese-bank-incident-root-cause-analysis-d076bf87a1e7) and the [Warp Finance attack](https://peckshield.medium.com/warpfinance-incident-root-cause-analysis-581a4869ee00).

Referenced code:
[DAO/LimboDAO.sol#L356](https://github.com/code-423n4/2022-01-behodler/blob/main/contracts/DAO/LimboDAO.sol#L356)
[DAO/LimboDAO.sol#L392](https://github.com/code-423n4/2022-01-behodler/blob/main/contracts/DAO/LimboDAO.sol#L392)

## Recommended Mitigation Steps

Use a fair pricing formula for the LP tokens, for example, the one proposed by [Alpha Finance](https://blog.alphafinance.io/fair-lp-token-pricing/).

