## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- LiquidityBasedTWAP

# [Vader TWAP averages wrong](https://github.com/code-423n4/2021-12-vader-findings/issues/148) 

# Handle

cmichel


# Vulnerability details

The vader price in `LiquidityBasedTWAP.getVaderPrice` is computed using the `pastLiquidityWeights` and `pastTotalLiquidityWeight` return values of the `syncVaderPrice`.

The `syncVaderPrice` function does not initialize all weights and the total liquidity weight does not equal the sum of the individual weights because it skips initializing the pair with the previous data if the TWAP update window has not been reached yet:

```solidity
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
        // @audit-info lastMeasurement is set in _updateVaderPrice to block.timestamp
        uint256 timeElapsed = block.timestamp - pairData.lastMeasurement;
        // @audit-info update period depends on pair
        // @audit-issue if update period not reached => does not initialize pastLiquidityWeights[i]
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

#### POC
This bug leads to several different issues. A big one is that an attacker can break the price functions and make them revert.
Observe what happens if an attacker calls `syncVaderPrice` twice in the same block:

- The first time any pairs that need to be updated are updated
- On the second call `_totalLiquidityWeight` is initialized to zero and all pairs have already been updated and thus skipped. `_totalLiquidityWeight` never increases and the storage variable `totalLiquidityWeight[uint256(Paths.VADER)] = _totalLiquidityWeight = 0;` is set to zero.
- DoS because calls to `getStaleVaderPrice` / `getVaderPrice` will revert in `_calculateVaderPrice` which divides by `totalLiquidityWeight = 0`.

Attacker keeps double-calling `syncVaderPrice` every time an update window of one of the pairs becomes eligible to be updated.


## Impact
This bug leads to using wrong averaging and ignoring entire pairs due to their weights being initialized to zero and never being changed if the update window is not met.
This in turn makes it easier to manipulate the price as potentially only a single pair needs to be price-manipulated.

It's also possible to always set the `totalLiquidityWeight` to zero by calling `syncVaderPrice` twice which in turn reverts all transactions making use of the price because of a division by zero in `_caluclateVaderPrice`.
An attacker can break the `USDV.mint` minting forever and any router calls to `VaderReserve.reimburseImpermanentLoss` also fail as they perform a call to the reverting price function.

## Recommended Mitigation Steps
Even if `timeElapsed < pairData.updatePeriod`, the old pair weight should still contribute to the total liquidity weight and be set in `pastLiquidityWeights`.
Move the `_totalLiquidityWeight += currentLiquidityEvaluation` and the `pastLiquidityWeights[i] = pastLiquidityEvaluation` assignments before the `continue`.


