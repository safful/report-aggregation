## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Wrong trading pricing calculations](https://github.com/code-423n4/2021-06-tracer-findings/issues/119) 

# Handle

0xsanson


# Vulnerability details

## Impact
In the Pricing contract, an agent can manipulate the trading prices by spamming an high amount of trades.

Indeed an agent can create an high amount of orders at an arbitrary price and with a near-zero amount (so the agent doesn't even need large funds); next he/she pairs the orders with another account and calls Trader.executeTrade; now every order calls a Pricing.recordTrade using the arbitrary price set by the agent.

Since the trades are all made in the same hour, by the way hourlyTracerPrices[currentHour] is calculated, it skews the average price towards the price set by the agent. This arbitrary value is used to calculate the fundingRates and the fairPrice, letting a malicious agent get the ability to manipulate the market.

## Proof of Concept
https://github.com/code-423n4/2021-06-tracer/blob/main/src/contracts/Pricing.sol#L129

## Tools Used
Manual analysis

## Recommended Mitigation Steps
Pass the fillAmount parameter to recordTrade(...), and calculate hourlyTracerPrices[currentHour].trades summing fillAmount instead of 1 every trade.

