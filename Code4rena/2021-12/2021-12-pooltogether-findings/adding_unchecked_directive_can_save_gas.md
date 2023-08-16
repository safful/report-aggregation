## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Adding unchecked directive can save gas](https://github.com/code-423n4/2021-12-pooltogether-findings/issues/111) 

# Handle

WatchPug


# Vulnerability details

For the arithmetic operations that will never over/underflow, using the unchecked directive (Solidity v0.8 has default overflow/underflow checks) can save some gas from the unnecessary internal over/underflow checks.

For example:

https://github.com/pooltogether/v4-periphery/blob/0e94c54774a6fce29daf9cb23353208f80de63eb/contracts/TwabRewards.sol#L153-L153

```solidity=153
uint256 _amount = _numberOfEpochs * _promotion.tokensPerEpoch;
```

`_numberOfEpochs` is uint8
`_promotion.tokensPerEpoch` is uint216

`_numberOfEpochs * _promotion.tokensPerEpoch` will never overflow.

