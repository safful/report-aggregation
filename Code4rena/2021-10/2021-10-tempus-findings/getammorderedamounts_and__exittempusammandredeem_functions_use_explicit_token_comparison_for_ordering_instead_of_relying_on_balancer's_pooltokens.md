## Tags

- bug
- sponsor confirmed
- 0 (Non-critical)
- resolved

# [getAMMOrderedAmounts and _exitTempusAmmAndRedeem functions use explicit token comparison for ordering instead of relying on Balancer's PoolTokens](https://github.com/code-423n4/2021-10-tempus-findings/issues/37) 

# Handle

hyh


# Vulnerability details

## Vulnerability Details
getAMMOrderedAmounts, https://github.com/code-423n4/2021-10-tempus/blob/main/contracts/TempusController.sol#L692, and _exitTempusAmmAndRedeem, https://github.com/code-423n4/2021-10-tempus/blob/main/contracts/TempusController.sol#L644, functions use explicit token comparison for ordering, while it is based on current Balancer pool implementation, which can change, leading to contract logic discrepancies.

In the same time _getAMMDetailsAndEnsureInitialized (https://github.com/code-423n4/2021-10-tempus/blob/main/contracts/TempusController.sol#L673) do rely on PoolTokens, which obtain token list in Balancer's call sequence as follows:
PoolTokens._getPoolTokens -> TwoTokenPoolsBalance._getTwoTokenPoolTokens -> TwoTokenPoolsBalance._getTwoTokenPoolBalances -> TwoTokenPoolsBalance._twoTokenPoolTokens[].

TwoTokenPoolsBalance._twoTokenPoolTokens[] is ordered during _registerTwoTokenPoolTokens, but this is current implementation.

It is safer to use vault.getPoolTokens(poolId) in getAMMOrderedAmounts to obtain an ordered pair.

This can matter as AMM token usage isn't symmetric (https://github.com/code-423n4/2021-10-tempus/blob/main/contracts/TempusController.sol#L84).

## Impact
Probability here is low and risk rating is minimal, but the impact can vary as TempusController contract  logic rely on token ordering.

