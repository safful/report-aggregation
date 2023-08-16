## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- Oracles

# [Exchange rates from Compound are assumed with 18 decimals](https://github.com/code-423n4/2021-08-yield-findings/issues/38) 

# Handle

shw


# Vulnerability details

## Impact

The `CTokenMultiOracle` contract assumes the exchange rates (borrowing rate) of Compound always have 18 decimals, while, however, which is not true. According to the [Compound documentation](https://compound.finance/docs/ctokens#exchange-rate), the exchange rate returned from the `exchangeRateCurrent` function is scaled by `1 * 10^(18 - 8 + Underlying Token Decimals)` (and so does `exchangeRateStored`). Using a wrong decimal number on the exchange rate could cause incorrect pricing on tokens.

## Proof of Concept

Referenced code:
[CTokenMultiOracle.sol#L110](https://github.com/code-423n4/2021-08-yield/blob/main/contracts/oracles/compound/CTokenMultiOracle.sol#L110)

## Recommended Mitigation Steps

Follow the documentation and get the decimals of the underlying tokens to set the correct decimal of a `Source`.

