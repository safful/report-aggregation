## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [range fee growth underflow](https://github.com/code-423n4/2021-09-sushitrident-2-findings/issues/25) 

# Handle

broccoli


# Vulnerability details

# range fee growth underflow
## Impact
The function `RangeFeeGrowth` [ConcentratedLiquidityPool.sol#L601-L633](https://github.com/sushiswap/trident/blob/c405f3402a1ed336244053f8186742d2da5975e9/contracts/pool/concentrated/ConcentratedLiquidityPool.sol#L601-L633) would revert the transaction in some cases.

When a pool cross a tick, it only updates either `feeGrowthOutside0` or `feeGrowthOutside1`. [Ticks.sol#L23-L53](https://github.com/sushiswap/trident/blob/c405f3402a1ed336244053f8186742d2da5975e9/contracts/libraries/concentratedPool/Ticks.sol#L23-L53)

`RangeFeeGrowth` calculates the fee as follow:
```solidity
            feeGrowthInside0 = _feeGrowthGlobal0 - feeGrowthBelow0 - feeGrowthAbove0;
            feeGrowthInside1 = _feeGrowthGlobal1 - feeGrowthBelow1 - feeGrowthAbove1;
```

`feeGrowthBelow + feeGrowthAbove` is not necessary smaller than `_feeGrowthGlobal`. Please see `POC`.


Users can not provide liquidity or burn liquidity. Fund will get stocked in the contract. I consider this is a high-risk issue.

## Proof of Concept
```python
    # This is the wrapper.
    # def add_liquidity(pool, amount, lower, upper)
    # def swap(pool, buy, amount)

    add_liquidity(pool, deposit_amount, -800, 500)
    add_liquidity(pool, deposit_amount, 400, 700)
    # We cross the tick here to trigger the bug.

    swap(pool, False, deposit_amount)
    # Only tick 700's feeGrowthOutside1 is updated

    swap(pool, True, deposit_amount)
    # Only tick 500's feeGrowthOutside0 is updated
    
    # current tick at -800

    # this would revert
    # feeGrowthBelow1 = feeGrowthGlobal1
    # feeGrowthGlobal1 - feeGrowthBelow1 - feeGrowthAbove1 would revert
    # user would not be able to mint/withdraw/cross this tick. The pool is broken
    add_liquidity(pool, deposit_amount, 400, 700)
```
## Tools Used
Hardhat

## Recommended Mitigation Steps

It's either modify the tick's algo or `RangeFeeGrowth`. The quick-fix I come up with is to deal with the fee in `RangeFeeGrowth`. However, I recommend the team to go through tick's logic again.

