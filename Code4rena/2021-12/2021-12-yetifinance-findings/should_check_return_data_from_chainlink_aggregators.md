## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [SHOULD CHECK RETURN DATA FROM CHAINLINK AGGREGATORS](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/91) 

# Handle

defsec


# Vulnerability details

## Impact

The latestRoundData function in the contract PriceFeed.sol fetches the asset price from a Chainlink aggregator using the latestRoundData function. However, there are no checks on roundID.

Stale prices could put funds at risk. According to Chainlink's documentation, This function does not error if no answer has been reached but returns 0, causing an incorrect price fed to the PriceOracle. The external Chainlink oracle, which provides index price information to the system, introduces risk inherent to any dependency on third-party data sources. For example, the oracle could fall behind or otherwise fail to be maintained, resulting in outdated data being fed to the index price calculations of the liquidity.

Example Medium Issue : https://github.com/code-423n4/2021-08-notional-findings/issues/18

## Proof of Concept

1. Navigate to the following contract.

"https://github.com/code-423n4/2021-12-yetifinance/blob/1da782328ce4067f9654c3594a34014b0329130a/packages/contracts/contracts/PriceFeed.sol#L578"

2. Only the following checks are implemented.

```
        if (!_response.success) {return true;}
        // Check for an invalid roundId that is 0
        if (_response.roundId == 0) {return true;}
        // Check for an invalid timeStamp that is 0, or in the future
        if (_response.timestamp == 0 || _response.timestamp > block.timestamp) {return true;}
        // Check for non-positive price
        if (_response.answer <= 0) {return true;}


```

## Tools Used

Manual Review

## Recommended Mitigation Steps

Consider to add checks on the return data with proper revert messages if the price is stale or the round is incomplete, for example:

```
(uint80 roundID, int256 price, , uint256 timeStamp, uint80 answeredInRound) = ETH_CHAINLINK.latestRoundData();
require(price > 0, "Chainlink price <= 0"); 
require(answeredInRound >= roundID, "...");
require(timeStamp != 0, "...");
```

