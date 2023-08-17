## Tags

- bug
- 3 (High Risk)
- disagree with severity
- sponsor confirmed
- valid

# [ORACLE DATA FEED CAN BE OUTDATED YET USED ANYWAYS WHICH WILL IMPACT ON PAYMENT LOGIC](https://github.com/code-423n4/2022-07-juicebox-findings/issues/138) 

# Lines of code

https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/828bf2f3e719873daa08081cfa0d0a6deaa5ace5/contracts/JBChainlinkV3PriceFeed.sol#L44
https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/828bf2f3e719873daa08081cfa0d0a6deaa5ace5/contracts/JBPrices.sol#L57
https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/828bf2f3e719873daa08081cfa0d0a6deaa5ace5/contracts/JBSingleTokenPaymentTerminalStore.sol#L387
https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/828bf2f3e719873daa08081cfa0d0a6deaa5ace5/contracts/JBSingleTokenPaymentTerminalStore.sol#L585
https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/828bf2f3e719873daa08081cfa0d0a6deaa5ace5/contracts/JBSingleTokenPaymentTerminalStore.sol#L661
https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/828bf2f3e719873daa08081cfa0d0a6deaa5ace5/contracts/JBSingleTokenPaymentTerminalStore.sol#L830
https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/828bf2f3e719873daa08081cfa0d0a6deaa5ace5/contracts/JBSingleTokenPaymentTerminalStore.sol#L868


# Vulnerability details

## Impact
The current implementation of `JBChainlinkV3PriceFeed` is used by the protocol to showcase how the feed will be retrieved via Chainlink Data Feeds. The feed is used to retrieve the `currentPrice`, which is also used afterwards by `JBPrices.priceFor()`, then by `JBSingleTokenPaymentTerminalStore.recordPaymentFrom()`, `JBSingleTokenPaymentTerminalStore.recordDistributionFor`, `JBSingleTokenPaymentTerminalStore.recordUsedAllowanceOf`, `JBSingleTokenPaymentTerminalStore._overflowDuring` and `JBSingleTokenPaymentTerminalStore._currentTotalOverflowOf`.
Although the current feeds are calculated by a non implemented IJBPriceFeed, if the implementation of the price feed is the same as the showcased in`JBChainlinkV3PriceFeed`, the retrieved data can be outdated or out of bounds.

It is important to remember that the sponsor said on the dedicated Discord Channel that also oracle pricing and data retrieval is inside the scope.


## Proof of Concept
Chainlink classifies their data feeds into four different groups regarding how reliable is each source thus, how risky they are. The groups are _Verified Feeds, Monitored Feeds, Custom Feeds and Specialized Feeds_ (they can be seen [here](https://docs.chain.link/docs/selecting-data-feeds/#data-feed-categories)). The risk is the lowest on the first one and highest on the last one.

A strong reliance on the price feeds has to be also monitored as recommended on the [Risk Mitigation section](https://docs.chain.link/docs/selecting-data-feeds/#risk-mitigation). There are several reasons why a data feed may fail such as unforeseen market events, volatile market conditions, degraded performance of infrastructure, chains, or networks, upstream data providers outage, malicious activities from third parties among others.

Chainlink recommends using their data feeds along with some controls to prevent mismatches with the retrieved data. Along some recommendations, the feed can include circuit breakers (for extreme price events), contract update delays (to ensure that the injected data into the protocol is fresh enough), manual kill-switches (to cease connection in case of found bug or vulnerability in an upstream contract), monitoring (control the deviation of the data) and soak testing (of the price feeds).

The `feed.lastRoundData()` interface parameters [according to Chainlink](https://docs.chain.link/docs/price-feeds-api-reference/) are the following:

    function latestRoundData() external view
        returns (
            uint80 roundId,             //  The round ID.
            int256 answer,              //  The price.
            uint256 startedAt,          //  Timestamp of when the round started.
            uint256 updatedAt,          //  Timestamp of when the round was updated.
            uint80 answeredInRound      //  The round ID of the round in which the answer was computed.
        )

Regarding Juicebox itself, only the `answer` is used on the `JBChainlinkV3PriceFeed.currentPrice()` implementation. The retrieved price of the `priceFeed` can be outdated and used anyways as a valid data because no timestamp tolerance of the update source time is checked while storing the return parameters of `feed.latestRoundData()` inside `JBChainlinkV3PriceFeed.currentPrice()` as recommended by Chainlink in [here](https://docs.chain.link/docs/using-chainlink-reference-contracts/#check-the-timestamp-of-the-latest-answer). The usage of outdated data can impact on how the Payment terminals work regarding pricing calculation and value measurement.

Precisely the following protocol logic within `JBSingleTokenPaymentTerminalStore​‌` will work unexpectedly regarding value management.

- `recordPaymentFrom()`:

  This function handles the minting of a project tokens according to a data source if one is given. If the retrieved value of the oracle is outdated, the `_weightRatio` at [Line 387](https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/828bf2f3e719873daa08081cfa0d0a6deaa5ace5/contracts/JBSingleTokenPaymentTerminalStore.sol#L387) will return an incorrect value and then the `tokenCount` calculated amount will suffer from this mismatch, impacting in the amount of tokens minted.

- `recordDistributionFor()`:

  Performs the recording of recently distributed funds for a project. On [line 580](https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/828bf2f3e719873daa08081cfa0d0a6deaa5ace5/contracts/JBSingleTokenPaymentTerminalStore.sol#L580) the `distributedAmount` is computed and if the boolean check is false, then the call will perform a call to `priceFor` at [line 585](https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/828bf2f3e719873daa08081cfa0d0a6deaa5ace5/contracts/JBSingleTokenPaymentTerminalStore.sol#L585). If the returned oracle value is not adjusted with current market prices, the `distributedAmount` will also drag that error computing an incorrect `distributedAmount`. Afterwards, because the `distributedAmount` is also used to update the token balances of the `msg.sender` ([line 598](https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/828bf2f3e719873daa08081cfa0d0a6deaa5ace5/contracts/JBSingleTokenPaymentTerminalStore.sol#L598)) it means that the mismatch impacts on the modified balance.

- `recordUsedAllowanceOf()`:

  Keeps record of used allowances of a project. It returns are analogue to the ones shown at `recordDistributionFor` where the `usedAmount` resembles the `distributedAmount`. The `usedAmount` is also used to update the project's balance. If the data of the oracle is outdated, the `usedAmount` will be calculated dragging that error.

- `_overflowDuring()`:

  Used to get the amount that is overflowing relative to a specified cycle. The data retrieved from the oracle is used to calculate the value of `_distributionLimitRemaining` on [line 827](https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/828bf2f3e719873daa08081cfa0d0a6deaa5ace5/contracts/JBSingleTokenPaymentTerminalStore.sol#L827) which is used later to calculate the return value if the boolean check performed at line 834 is true. Because the return of this function is the current balance of a project minus the amount that can be still distributed, if the amount that can still be distributed is wrong so will be the subtraction thus the return value.

- `_currentTotalOverflowOf()`:

  Similar to the latter but used to get the overflow of all the terminals of a project. If the retrieved data has a mismatch with the market, the `_totalOverflow18Decimal` calculated on [line 866](https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/828bf2f3e719873daa08081cfa0d0a6deaa5ace5/contracts/JBSingleTokenPaymentTerminalStore.sol#L827) if the boolean check is false will drag this mismatch which will also be dragged into the final return of the function.

The issues of those miscalculations impact on every project currently minted, which also affects subsequently on each user that has tokens of a project resulting in a high reach impact.


## Recommended Mitigation Steps

As Chainlink [recommends](https://docs.chain.link/docs/using-chainlink-reference-contracts/#check-the-timestamp-of-the-latest-answer):

> Your application should track the `latestTimestamp` variable or use the `updatedAt` value from the `latestRoundData()` function to make sure that the latest answer is recent enough for your application to use it. If your application detects that the reported answer is not updated within the heartbeat or within time limits that you determine are acceptable for your application, pause operation or switch to an alternate operation mode while identifying the cause of the delay.

> During periods of low volatility, the heartbeat triggers updates to the latest answer. Some heartbeats are configured to last several hours, so your application should check the timestamp and verify that the latest answer is recent enough for your application.

It is recommended both to add also a tolerance that compares the `updatedAt` return timestamp from `latestRoundData()` with the current block timestamp and ensure that the `priceFeed` is being updated with the required frequency.

If the `ETH/USD` is the only one that is needed to retrieve, because it is the most popular and available pair it can also be useful to add other oracle to get the price feed (such as Uniswap's). This can be used as a redundancy in the case of having one oracle that returns outdated values (what is outdated and what is up to date can be determined by a tolerance as mentioned).

