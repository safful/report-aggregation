## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Fee not decayed if past `decayTime`](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/204) 

# Handle

cmichel


# Vulnerability details

The `ThreePieceWiseLinearPriceCurve.calculateDecayedFee` function is supposed to decay the `lastFeePercent` over time.
This is correctly done in the `decay > 0 && decay < decayTime` case, but for the `decay > decayTime` case it does not decay at all but should set it to 0 instead..

```solidity
if (decay > 0 && decay < decayTime) {
    // @audit if decay is close to decayTime, this fee will be zero. but below it'll be 1. the more time passes, the higher the decay. but then decay > decayTime should return 0.
    fee = lastFeePercent.sub(lastFeePercent.mul(decay).div(decayTime));
} else {
    fee = lastFeePercent;
}
```

## Recommended Mitigation Steps
It seems wrong to handle the `decay == 0` case (decay happened in same block) the same way as the `decay >= decayTime` case (decay happened long time ago) as is done in the `else` branch.
I believe it should be like this instead:

```solidity
// decay == 0 case should be full lastFeePercent
if(decay < decayTime) {
    fee = lastFeePercent.sub(lastFeePercent.mul(decay).div(decayTime));
} else {
    // reset to zero if decay >= decayTime
    fee = 0;
}
```

