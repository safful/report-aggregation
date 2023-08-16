## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Missing `maltDataLab.trackReserveRatio()` in some cases after `swingTrader.sellMalt()`](https://github.com/code-423n4/2021-11-malt-findings/issues/320) 

# Handle

WatchPug


# Vulnerability details

Based on the context, `maltDataLab.trackReserveRatio()` should be called once a market buy/sell is made.

However, in `_distributeSupply()` when `swingAmount >= tradeSize`, after a market sell, the function returned without `maltDataLab.trackReserveRatio()`.

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/StabilizerNode.sol#L145-L174

```solidity=145{168,170,172}
  function stabilize() external notSameBlock {
    auction.checkAuctionFinalization();

    require(
      block.timestamp >= stabilizeWindowEnd || _stabilityWindowOverride(),
      "Can't call stabilize"
    );
    stabilizeWindowEnd = block.timestamp + stabilizeBackoffPeriod;

    rewardThrottle.checkRewardUnderflow();

    uint256 exchangeRate = maltDataLab.maltPriceAverage(priceAveragePeriod);

    if (!_shouldAdjustSupply(exchangeRate)) {
      maltDataLab.trackReserveRatio();

      lastStabilize = block.timestamp;
      return;
    }

    emit Stabilize(block.timestamp, exchangeRate);

    if (exchangeRate > maltDataLab.priceTarget()) {
      _distributeSupply();
    } else {
      _startAuction();
    }

    lastStabilize = block.timestamp;
  }
```

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/StabilizerNode.sol#L211-L246

```solidity=211{228-230,244}
  function _distributeSupply() internal {
    if (supplyDistributionController != address(0)) {
      bool success = ISupplyDistributionController(supplyDistributionController).check();
      if (!success) {
        return;
      }
    }

    uint256 priceTarget = maltDataLab.priceTarget();
    uint256 tradeSize = dexHandler.calculateMintingTradeSize(priceTarget).div(expansionDampingFactor);

    if (tradeSize == 0) {
      return;
    }

    uint256 swingAmount = swingTrader.sellMalt(tradeSize); // @Auditor: At this time, a market operation occurred, affecting the reserveRatio

    if (swingAmount >= tradeSize) {
      return;
    }

    tradeSize = tradeSize - swingAmount;

    malt.mint(address(dexHandler), tradeSize);
    emit MintMalt(tradeSize);
    uint256 rewards = dexHandler.sellMalt();

    auctionBurnReserveSkew.addAbovePegObservation(tradeSize);

    uint256 remaining = _replenishLiquidityExtension(rewards);

    _distributeRewards(remaining);

    maltDataLab.trackReserveRatio();
    impliedCollateralService.claim();
  }
```

### Recommendation

Consider moving `maltDataLab.trackReserveRatio()` from `_distributeSupply()`, `_startAuction()` to `stabilize()` before L173.

