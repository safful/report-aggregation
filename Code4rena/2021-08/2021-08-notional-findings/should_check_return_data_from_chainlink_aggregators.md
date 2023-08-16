## Tags

- bug
- duplicate
- 2 (Med Risk)
- sponsor confirmed

# [SHOULD CHECK RETURN DATA FROM CHAINLINK AGGREGATORS](https://github.com/code-423n4/2021-08-notional-findings/issues/34) 

# Handle

defsec


# Vulnerability details

## Impact

The latestRoundData function in the contract ExchangeRate.sol fetches the asset price from a Chainlink aggregator using the latestRoundData function. However, there are no checks on roundID nor timeStamp, resulting in stale prices. Stale prices could put funds at risk. Freshness of the returned price should be checked, since it affects an account's health (and therefore liquidations). Stale prices that do not reflect the current market price anymore could be used which would influence the liquidation pricing.


## Proof of Concept

1. Navigate to "https://github.com/code-423n4/2021-08-notional/blob/4b51b0de2b448e4d36809781c097c7bc373312e9/contracts/internal/valuation/ExchangeRate.sol".
2. latestRoundData has been used in the repository.
3. The code can be seen from the below. Stale prices have not been checked.

```
        } else {
            address rateOracle = address(bytes20(data << 96));
            // prettier-ignore
            (
                /* uint80 */,
                rate,
                /* uint256 */,
                /* uint256 */,
                /* uint80 */
            ) = AggregatorV2V3Interface(rateOracle).latestRoundData();
            require(rate > 0, "ExchangeRate: invalid rate");

            uint8 rateDecimalPlaces = uint8(bytes1(data << 88));
            rateDecimals = int256(10**rateDecimalPlaces);
            if (
                bytes1(data << 80) != Constants.BOOL_FALSE /* mustInvert */
            ) {
                rate = rateDecimals.mul(rateDecimals).div(rate);
            }
        }

```

## Tools Used

## Recommended Mitigation Steps

Consider to add checks on the return data with proper revert messages if the price is stale or the round is incomplete, for example:

```
(uint80 roundID, int256 price, , uint256 timeStamp, uint80 answeredInRound) = ETH_CHAINLINK.latestRoundData();
require(price > 0, "Chainlink price <= 0"); 
require(answeredInRound >= roundID, "...");
require(timeStamp != 0, "...");
```


