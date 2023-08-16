## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [Sandwich attack on astroport sweep](https://github.com/code-423n4/2022-02-anchor-findings/issues/33) 

# Lines of code

https://github.com/code-423n4/2022-02-anchor/blob/7af353e3234837979a19ddc8093dc9ad3c63ab6b/contracts%2Fanchor-token-contracts%2Fcontracts%2Fcollector%2Fsrc%2Fcontract.rs#L130-L137


# Vulnerability details

## Impact
The collector contract allows anyone to `sweep`, swapping an asset token to ANC through astro port.
Note that `belief_price` is not set and `config.max_spread` might not be set as well or misconfigured.

This allows an attacker to create a contract to perform a sandwich attack to make a profit on this trade.

> A common attack in DeFi is the sandwich attack. Upon observing a trade of asset X for asset Y, an attacker frontruns the victim trade by also buying asset Y, lets the victim execute the trade, and then backruns (executes after) the victim by trading back the amount gained in the first trade. Intuitively, one uses the knowledge that someone’s going to buy an asset, and that this trade will increase its price, to make a profit. The attacker’s plan is to buy this asset cheap, let the victim buy at an increased price, and then sell the received amount again at a higher price afterwards.

Trades can happen at a bad price and lead to receiving fewer tokens than at a fair market price.
The attacker's profit is the protocol's loss.

#### POC
Attacker creates a contract that triggers 3 messages for the sandwich attack:

- Astroport: buy ANC with asset
- call `sweep` which trades at bad price
- Astroport: sell assets from the first message for profit


## Recommended Mitigation Steps
Consider setting a ANC/asset `belief_price` from an oracle.


