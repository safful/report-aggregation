## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [[NAZ-M1] Chainlink's `latestRoundData` Might Return Stale Results](https://github.com/code-423n4/2022-08-olympus-findings/issues/441) 

# Lines of code

https://github.com/code-423n4/2022-08-olympus/blob/main/src/modules/PRICE.sol#L161
https://github.com/code-423n4/2022-08-olympus/blob/main/src/modules/PRICE.sol#L170


# Vulnerability details

## Impact
Across these contracts, you are using Chainlink's `latestRoundData` API, but there is only a check on `updatedAt`. This could lead to stale prices according to the Chainlink documentation:

* [Historical Price data](https://docs.chain.link/docs/historical-price-data/#historical-rounds)
* [Checking Your returned answers](https://docs.chain.link/docs/faq/#how-can-i-check-if-the-answer-to-a-round-is-being-carried-over-from-a-previous-round)

The result of `latestRoundData` API will be used across various functions, therefore, a stale price from Chainlink can lead to loss of funds to end-users.

## Tools Used
Manual Review

## Recommended Mitigation Steps
Consider adding the missing checks for stale data.

For example:
```js
(uint80 roundID ,answer,, uint256 timestamp, uint80 answeredInRound) = AggregatorV3Interface(chainLinkAggregatorMap[underlying]).latestRoundData();

require(answer > 0, "Chainlink price <= 0"); 
require(answeredInRound >= roundID, "Stale price");
require(timestamp != 0, "Round not complete");
```