## Tags

- bug
- duplicate
- 2 (Med Risk)
- sponsor confirmed

# [Missing validation on latestRoundData](https://github.com/code-423n4/2021-08-notional-findings/issues/92) 

# Handle

a_delamo


# Vulnerability details

On `ExchangeRate.sol`, we are using `latestRoundData`, but there are no validations that the data is not stale.

The current code is:

```solidity
            (
                /* uint80 */,
                rate,
                /* uint256 */,
                /* uint256 */,
                /* uint80 */
            ) = AggregatorV2V3Interface(rateOracle).latestRoundData();
            require(rate > 0, "ExchangeRate: invalid rate");
```

But is missing the checks to validate the data is stale 

```solidity
(roundId, rawPrice,, updatedAt, answeredInRound) = AggregatorV2V3Interface(rateOracle).latestRoundData();
require(rawPrice > 0, "Chainlink price <= 0");
require(updateTime != 0, "Incomplete round");
require(answeredInRound >= roundId, "Stale price");
```

More information: https://docs.chain.link/docs/faq/#how-can-i-check-if-the-answer-to-a-round-is-being-carried-over-from-a-previous-round



