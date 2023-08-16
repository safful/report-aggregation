## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Usage of deprecated Chainlink functions](https://github.com/code-423n4/2022-03-rolla-findings/issues/17) 

# Lines of code

https://github.com/code-423n4/2022-03-rolla/blob/main/quant-protocol/contracts/pricing/oracle/ChainlinkOracleManager.sol#L120
https://github.com/code-423n4/2022-03-rolla/blob/main/quant-protocol/contracts/pricing/oracle/ChainlinkFixedTimeOracleManager.sol#L81
https://github.com/code-423n4/2022-03-rolla/blob/main/quant-protocol/contracts/pricing/oracle/ChainlinkFixedTimeOracleManager.sol#L84


# Vulnerability details

## Impact
The Chainlink functions `latestAnswer()` and `getAnswer()` are deprecated. Instead, use the [`latestRoundData()`](https://docs.chain.link/docs/price-feeds-api-reference/#latestrounddata) and [`getRoundData()`](https://docs.chain.link/docs/price-feeds-api-reference/#getrounddata) functions.

## Proof of Concept
https://github.com/code-423n4/2022-03-rolla/blob/main/quant-protocol/contracts/pricing/oracle/ChainlinkOracleManager.sol#L120

https://github.com/code-423n4/2022-03-rolla/blob/main/quant-protocol/contracts/pricing/oracle/ChainlinkFixedTimeOracleManager.sol#L81

https://github.com/code-423n4/2022-03-rolla/blob/main/quant-protocol/contracts/pricing/oracle/ChainlinkFixedTimeOracleManager.sol#L84

Go to https://etherscan.io/address/0x5f4eC3Df9cbd43714FE2740f5E3616155c5b8419#code and search for `latestAnswer()` or `getAnswer()`. You'll find the deprecation notice.

## Tools Used
none

## Recommended Mitigation Steps
Switch to `latestRoundData()` as described [here](https://docs.chain.link/docs/price-feeds-api-reference/#latestrounddata)

