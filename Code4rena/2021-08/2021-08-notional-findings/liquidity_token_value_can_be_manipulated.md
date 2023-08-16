## Tags

- bug
- 3 (High Risk)
- disagree with severity
- sponsor confirmed

# [Liquidity token value can be manipulated](https://github.com/code-423n4/2021-08-notional-findings/issues/85) 

# Handle

cmichel


# Vulnerability details

## Vulnerability Details
The liquidity token value (`AssetHandler.getLiquidityTokenValue`) is the sum of the value of the individual claims on cash (underlying or rather cTokens) and fCash.
The amount to redeem on each of these is computed as the LP token to redeem relative to the total LP tokens, see `AssetHandler.getCashClaims` / `AssetHandler.getHaircutCashClaims`:

```solidity
// @audit token.notional are the LP tokens to redeem
assetCash = market.totalAssetCash.mul(token.notional).div(market.totalLiquidity);
fCash = market.totalfCash.mul(token.notional).div(market.totalLiquidity);
```

This means the value depends on the **current market reserves** which can be manipulated.
You're essentially computing a spot price (even though the individual values use a TWAP price) because you use the current market reserves which can be manipulated.

See the "How do I tell if I’m using spot price?" section on [https://shouldiusespotpriceasmyoracle.com/](https://shouldiusespotpriceasmyoracle.com/).
> However, by doing this you’re actually incorporating the spot price because you’re still dependent on the reserve balances of the pool. This is an extremely subtle detail, and more than one project has been caught by it. You can read more about this [footgun](https://cmichel.io/pricing-lp-tokens/) in this writeup by @cmichelio.

## Impact
The value of an LP token is computed as `assetCashClaim + assetRate.convertFromUnderlying( presentValue(fCashClaim) )` where `(assetCashClaim, fCashClaim)` depend on the current market reserves which can be manipulated by an attacker via flashloans.
Therefore, an attacker trading large amounts in the market either increases or decreases the value of an LP token.

If the value decreases, they can try to liquidate users borrowing against their LP tokens / nTokens.
If the value increases, they can borrow against it and potentially receive an under-collateralized borrow this way, making a profit.

The exact profitability of such an attack depends on the AMM as the initial reserve manipulation and restoring the reserves later incurs fees and slippage.
In constant-product AMMs like Uniswap it's profitable and several projects have already been exploited by this, like [warp.finance](https://cmichel.io/pricing-lp-tokens/).
However, Notional Finance uses a more complicated AMM and the contest was too short for me to do a more thorough analysis. It seems like a similar attack could be possible here as described by the developers when talking about a different context of using TWAP oracles:
> "Oracle rate protects against short term price manipulation. Time window will be set to a value on the order of minutes to hours. This is to protect fCash valuations from market manipulation. For example, a trader could use a flash loan to dump a large amount of cash into the market and depress interest rates. Since we value fCash in portfolios based on these rates, portfolio values will decrease and they may then be liquidated." - Market.sol L424

## Recommendation
Do not use the current market reserves to determine the value of LP tokens.
Think about how to implement a TWAP oracle for the LP tokens themselves, instead of combining it from the two TWAPs of the claimables.


