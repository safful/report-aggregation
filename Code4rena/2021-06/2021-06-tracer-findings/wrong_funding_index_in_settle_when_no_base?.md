## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Wrong funding index in settle when no base?](https://github.com/code-423n4/2021-06-tracer-findings/issues/106) 

# Handle

cmichel


# Vulnerability details

The `TracerPerpetualSwaps.settle` function updates the user's last index to `currentGlobalFundingIndex`, however a comment states:

> "// Note: global rates reference the last fully established rate (hence the -1), and not the current global rate. User rates reference the last saved user rate"

The code for the `else` branch also updates the last index to `currentGlobalFundingIndex - 1` instead of `currentGlobalFundingIndex`.

```solidity
if (accountBalance.position.base == 0) {
    // set to the last fully established index
    // @audit shouldn't this be global - 1 like below?
    accountBalance.lastUpdatedIndex = currentGlobalFundingIndex;
    accountBalance.lastUpdatedGasPrice = IOracle(gasPriceOracle).latestAnswer();
}
```


## Impact
It might be possible that first-time depositors skip having to pay the first funding rate period as the `accountLastUpdatedIndex + 1 < currentGlobalFundingIndex` check will still return `false` when the funding rates are updated the next time.

## Recommended Mitigation Steps
Check if setting it to `currentGlobalFundingIndex` or to `currentGlobalFundingIndex - 1` is correct.

