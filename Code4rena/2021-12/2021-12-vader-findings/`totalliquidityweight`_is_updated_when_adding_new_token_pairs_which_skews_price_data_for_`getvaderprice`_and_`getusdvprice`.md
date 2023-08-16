## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- LiquidityBasedTWAP

# [`totalLiquidityWeight` Is Updated When Adding New Token Pairs Which Skews Price Data For `getVaderPrice` and `getUSDVPrice`](https://github.com/code-423n4/2021-12-vader-findings/issues/105) 

# Handle

leastwood


# Vulnerability details

## Impact

The `_addVaderPair` function is called by the `onlyOwner` role. The relevant data in the `twapData` mapping is set by querying the respective liquidity pool and Chainlink oracle. `totalLiquidityWeight` for the `VADER` path is also incremented by the `pairLiquidityEvaluation` amount (calculated within `_addVaderPair`). If a user then calls `syncVaderPrice`, the recently updated `totalLiquidityWeight` will be taken into consideration when iterating through all token pairs eligible for price updates to calculate the liquidity weight for each token pair. This data is stored in `pastTotalLiquidityWeight` and `pastLiquidityWeights` respectively.

As a result, newly added token pairs will increase `pastTotalLiquidityWeight` while leaving `pastLiquidityWeights` underrepresented. This only occurs if `syncVaderPrice` is called before the update period for the new token has not been passed.

This issue also affects how the price for `USDV` is synced.

## Proof of Concept

https://github.com/code-423n4/2021-12-vader/blob/main/contracts/lbt/LiquidityBasedTWAP.sol#L299
```
function _addVaderPair(
    IUniswapV2Pair pair,
    IAggregatorV3 oracle,
    uint256 updatePeriod
) internal {
    require(
        updatePeriod != 0,
        "LBTWAP::addVaderPair: Incorrect Update Period"
    );

    require(oracle.decimals() == 8, "LBTWAP::addVaderPair: Non-USD Oracle");

    ExchangePair storage pairData = twapData[address(pair)];

    bool isFirst = pair.token0() == vader;

    (address nativeAsset, address foreignAsset) = isFirst
        ? (pair.token0(), pair.token1())
        : (pair.token1(), pair.token0());

    oracles[foreignAsset] = oracle;

    require(nativeAsset == vader, "LBTWAP::addVaderPair: Unsupported Pair");

    pairData.foreignAsset = foreignAsset;
    pairData.foreignUnit = uint96(
        10**uint256(IERC20Metadata(foreignAsset).decimals())
    );

    pairData.updatePeriod = updatePeriod;
    pairData.lastMeasurement = block.timestamp;

    pairData.nativeTokenPriceCumulative = isFirst
        ? pair.price0CumulativeLast()
        : pair.price1CumulativeLast();

    (uint256 reserve0, uint256 reserve1, ) = pair.getReserves();

    (uint256 reserveNative, uint256 reserveForeign) = isFirst
        ? (reserve0, reserve1)
        : (reserve1, reserve0);

    uint256 pairLiquidityEvaluation = (reserveNative *
        previousPrices[uint256(Paths.VADER)]) +
        (reserveForeign * getChainlinkPrice(foreignAsset));

    pairData.pastLiquidityEvaluation = pairLiquidityEvaluation;

    totalLiquidityWeight[uint256(Paths.VADER)] += pairLiquidityEvaluation;

    vaderPairs.push(pair);

    if (maxUpdateWindow < updatePeriod) maxUpdateWindow = updatePeriod;
}
```

https://github.com/code-423n4/2021-12-vader/blob/main/contracts/lbt/LiquidityBasedTWAP.sol#L113-L148
```
function syncVaderPrice()
    public
    override
    returns (
        uint256[] memory pastLiquidityWeights,
        uint256 pastTotalLiquidityWeight
    )
{
    uint256 _totalLiquidityWeight;
    uint256 totalPairs = vaderPairs.length;
    pastLiquidityWeights = new uint256[](totalPairs);
    pastTotalLiquidityWeight = totalLiquidityWeight[uint256(Paths.VADER)];

    for (uint256 i; i < totalPairs; ++i) {
        IUniswapV2Pair pair = vaderPairs[i];
        ExchangePair storage pairData = twapData[address(pair)];
        uint256 timeElapsed = block.timestamp - pairData.lastMeasurement;

        if (timeElapsed < pairData.updatePeriod) continue;

        uint256 pastLiquidityEvaluation = pairData.pastLiquidityEvaluation;
        uint256 currentLiquidityEvaluation = _updateVaderPrice(
            pair,
            pairData,
            timeElapsed
        );

        pastLiquidityWeights[i] = pastLiquidityEvaluation;

        pairData.pastLiquidityEvaluation = currentLiquidityEvaluation;

        _totalLiquidityWeight += currentLiquidityEvaluation;
    }

    totalLiquidityWeight[uint256(Paths.VADER)] = _totalLiquidityWeight;
}
```

As shown above, `pastTotalLiquidityWeight = totalLiquidityWeight[uint256(Paths.VADER)]` loads in the total liquidity weight which is updated when `_addVaderPair` is called. However, `pastLiquidityWeights` is calculated by iterating through each token pair that is eligible to be updated.

## Tools Used

Manual code review.

## Recommended Mitigation Steps

Consider removing the line `totalLiquidityWeight[uint256(Paths.VADER)] += pairLiquidityEvaluation;` in `_addVaderPair` so that newly added tokens do not impact upcoming queries for `VADER/USDV` price data. This should ensure `syncVaderPrice` and `syncUSDVPrice` cannot be manipulated when adding new tokens.

