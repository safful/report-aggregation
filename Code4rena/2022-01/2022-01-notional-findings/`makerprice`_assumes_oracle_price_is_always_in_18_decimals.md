## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [`makerPrice` assumes oracle price is always in 18 decimals](https://github.com/code-423n4/2022-01-notional-findings/issues/198) 

# Handle

cmichel


# Vulnerability details

The `EIP1271Wallet._validateOrder` function computes a `makerPrice` which ends up in `takerAmount` decimals which are 18 decimals as `takerToken` is always `WETH`.
It is compared to the `priceFloor` return value from chainlink which must therefore also be in 18 decimals.
This seems to be the case for this old deprecated API but should be fixed and adjusted to use the oracle decimals if Chainlink is upgraded to the new API.

## Recommended Mitigation Steps
Upgrade and adjust the decimals of `makerPrice` to match `priceFloor` decimals.


