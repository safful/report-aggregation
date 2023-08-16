## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Can not update target price](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/143) 

# Handle

jonah1005


# Vulnerability details

## Impact
The sanity checks in `rampTargetPrice` are broken
[SwapUtils.sol#L1571-L1581](https://github.com/code-423n4/2021-11-bootfinance/blob/main/customswap/contracts/SwapUtils.sol#L1571-L1581)
```solidity
        if (futureTargetPricePrecise < initialTargetPricePrecise) {
            require(
                futureTargetPricePrecise.mul(MAX_RELATIVE_PRICE_CHANGE).div(WEI_UNIT) >= initialTargetPricePrecise,
                "futureTargetPrice_ is too small"
            );
        } else {
            require(
                futureTargetPricePrecise <= initialTargetPricePrecise.mul(MAX_RELATIVE_PRICE_CHANGE).div(WEI_UNIT),
                "futureTargetPrice_ is too large"
            );
        }
```
If `futureTargetPricePrecise` is smaller than `initialTargetPricePrecise` 0.01 of `futureTargetPricePrecise` would never larger than `initialTargetPricePrecise`.

Admin would not be able to ramp the target price. As it's one of the most important features of the customswap, I consider this is a high-risk issue

## Proof of Concept
Here's a web3.py script to demo that it's not possible to change the target price even by 1 wei.
```python
    p1, p2, _, _ =swap.functions.targetPriceStorage().call()
    future = w3.eth.getBlock(w3.eth.block_number)['timestamp'] + 200 * 24 * 3600

    # futureTargetPrice_ is too small
    swap.functions.rampTargetPrice(p1 -1, future).transact()
    # futureTargetPrice_ is too large
    swap.functions.rampTargetPrice(p1 + 1, future).transact()
```
## Tools Used
None
## Recommended Mitigation Steps

Would it be something like:
```solidity
        if (futureTargetPricePrecise < initialTargetPricePrecise) {
            require(
                futureTargetPricePrecise.mul(MAX_RELATIVE_PRICE_CHANGE + WEI_UNIT).div(WEI_UNIT) >= initialTargetPricePrecise,
                "futureTargetPrice_ is too small"
            );
        } else {
            require(
                futureTargetPricePrecise <= initialTargetPricePrecise.mul(MAX_RELATIVE_PRICE_CHANGE + WEI_UNIT).div(WEI_UNIT),
                "futureTargetPrice_ is too large"
            );
        }
```
I believe the dev would spot this mistake if there's a more relaxed timeline.

