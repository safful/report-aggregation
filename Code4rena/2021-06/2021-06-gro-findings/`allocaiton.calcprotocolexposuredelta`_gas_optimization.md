## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [`Allocaiton.calcProtocolExposureDelta` gas optimization](https://github.com/code-423n4/2021-06-gro-findings/issues/102) 

# Handle

cmichel


# Vulnerability details

## Vulnerability Details
`Allocaiton.calcProtocolExposureDelta` should break out of the loop to save gas after `protocolExposedDeltaUsd` is set.

```solidity
if (protocolExposedDeltaUsd == 0 && protocolExposure[i] > sysState.rebalanceThreshold) {
    // ...Calculate the delta between exposure and target
    uint256 target = sysState.rebalanceThreshold.sub(sysState.targetBuffer);
    protocolExposedDeltaUsd = protocolExposure[i].sub(target).mul(sysState.totalCurrentAssetsUsd).div(
        PERCENTAGE_DECIMAL_FACTOR
    );
    protocolExposedIndex = i;
    // @audit break here
}
```

