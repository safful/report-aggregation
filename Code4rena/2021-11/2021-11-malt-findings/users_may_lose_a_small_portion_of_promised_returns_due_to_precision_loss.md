## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Users may lose a small portion of promised returns due to precision loss](https://github.com/code-423n4/2021-11-malt-findings/issues/305) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/AuctionEscapeHatch.sol#L131-L140

```solidity=131
    uint256 progressionBps = (block.timestamp - auctionEndTime) * 10000 / cooloffPeriod;
    if (progressionBps > 10000) {
        progressionBps = 10000;
    }

    if (fullReturn > amount) {
        // Allow a % of profit to be realised
        uint256 maxProfit = (fullReturn - amount) * (maxEarlyExitBps * progressionBps / 10000) / 1000;
        return amount + maxProfit;
    }
```

If we assume that `maxEarlyExitBps` is 200 and `cooloffPeriod` is 1 day, when `progressionBps` less than 50, `(maxEarlyExitBps * progressionBps / 10000)` will be 0 due to precision loss, which resulted in `maxProfit` is 0.

When `maxEarlyExitBps` is set smaller, the margin of error will be even larger.

# POC

Given:

- Current price of arb token is 0.8 DAI

1. Alice calls `purchaseArbitrageTokens()` and purchase with 8,000 DAI;
2. 7 mins later, the market price of MALT become 0.9 DAI; Alice calls `exitEarly()`, it will mint 8,888.88 Malt and receive 8,000 DAI, while it's expected to 8,890 MALT and 8,000.96 DAI.

### Recommendation

Change to:

```solidity
if (fullReturn > amount) {
    // Allow a % of profit to be realised
    uint256 maxProfit = (fullReturn - amount) * (maxEarlyExitBps * 1000 * progressionBps / 10000) / 1000 / 1000;
    return amount + maxProfit;
}
```

