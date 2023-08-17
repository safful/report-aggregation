## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Chainlink pricer is using a deprecated API](https://github.com/code-423n4/2022-05-bunker-findings/issues/1) 

# Lines of code

https://github.com/bunkerfinance/bunker-protocol/blob/752126094691e7457d08fc62a6a5006df59bd2fe/contracts/PriceOracleImplementation.sol#L29-L30


# Vulnerability details

## Impact
According to Chainlink's documentation, the latestAnswer function is deprecated. This function might suddenly stop working if Chainlink stop supporting deprecated APIs. And the old API can return stale data.

## Proof of Concept
https://github.com/bunkerfinance/bunker-protocol/blob/752126094691e7457d08fc62a6a5006df59bd2fe/contracts/PriceOracleImplementation.sol#L29-L30
## Tools Used
None
## Recommended Mitigation Steps
Use the latestRoundData function to get the price instead. Add checks on the return data with proper revert messages if the price is stale or the round is uncomplete
https://docs.chain.link/docs/price-feeds-api-reference/

