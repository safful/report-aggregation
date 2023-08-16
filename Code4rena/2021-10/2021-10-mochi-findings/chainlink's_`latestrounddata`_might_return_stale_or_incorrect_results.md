## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Chainlink's `latestRoundData` might return stale or incorrect results](https://github.com/code-423n4/2021-10-mochi-findings/issues/87) 

# Handle

nikitastupin


# Vulnerability details

## Proof of Concept

https://github.com/code-423n4/2021-10-mochi/blob/8458209a52565875d8b2cefcb611c477cefb9253/projects/mochi-cssr/contracts/adapter/ChainlinkAdapter.sol#L49 

The `ChainlinkAdapter` calls out to a Chainlink oracle receiving the `latestRoundData()`. If there is a problem with Chainlink starting a new round and finding consensus on the new value for the oracle (e.g. Chainlink nodes abandon the oracle, chain congestion, vulnerability/attacks on the chainlink system) consumers of this contract may continue using outdated stale or incorrect data (if oracles are unable to submit no new round is started).

## Recommended Mitigation Steps

Add the following checks:

```
...
( roundId, rawPrice, , updateTime, answeredInRound ) = AggregatorV3Interface(XXXXX).latestRoundData();
require(rawPrice > 0, "Chainlink price <= 0");
require(updateTime != 0, "Incomplete round");
require(answeredInRound >= roundId, "Stale price");
...
```

## References

- https://consensys.net/diligence/audits/2021/09/fei-protocol-v2-phase-1/#chainlinkoraclewrapper-latestrounddata-might-return-stale-results
- https://github.com/code-423n4/2021-05-fairside-findings/issues/70

