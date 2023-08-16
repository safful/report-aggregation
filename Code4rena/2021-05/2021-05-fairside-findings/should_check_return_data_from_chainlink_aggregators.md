## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- resolved

# [Should check return data from Chainlink aggregators](https://github.com/code-423n4/2021-05-fairside-findings/issues/70) 

# Handle

shw


# Vulnerability details

## Impact

The `getEtherPrice` function in the contract `FSDNetwork` fetches the ETH price from a Chainlink aggregator using the `latestRoundData` function. However, there are no checks on `roundID` nor `timeStamp`, resulting in stale prices.

## Proof of Concept

Referenced code:
[FSDNetwork.sol#L376-L381](https://github.com/code-423n4/2021-05-fairside/blob/main/contracts/network/FSDNetwork.sol#L376-L381)

## Recommended Mitigation Steps

Add checks on the return data with proper revert messages if the price is stale or the round is uncomplete, for example:

```solidity
(uint80 roundID, int256 price, , uint256 timeStamp, uint80 answeredInRound) = ETH_CHAINLINK.latestRoundData();
require(answeredInRound >= roundID, "...");
require(timeStamp != 0, "...");
```

