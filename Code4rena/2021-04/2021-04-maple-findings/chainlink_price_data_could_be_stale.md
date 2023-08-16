## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- resolved

# [Chainlink Price data could be stale](https://github.com/code-423n4/2021-04-maple-findings/issues/82) 

# Handle

@cmichelio


# Vulnerability details

## Vulnerability Details

There is no check if the return value indicates stale data. This could lead to stale prices according to the Chainlink documentation:
  * ["if answeredInRound < roundId could indicate stale data."](https://docs.chain.link/docs/developer-communications#current-notifications)
  * ["A timestamp with zero value means the round is not complete and should not be used."](https://docs.chain.link/docs/historical-price-data#solidity)


## Impact

The price oracle might return unreliable price data which can lead to a variety of different issues in the protocol, for example, for liquidating more staker & lender tokens than required at fair market price.

## Recommended Mitigation Steps

Add missing checks for stale data. See example [here](https://github.com/cryptexfinance/contracts/blob/master/contracts/oracles/ChainlinkOracle.sol#L58-L65).


