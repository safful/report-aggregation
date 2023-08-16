## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed

# [Wrong token allocation computation for token decimals != 18 if floor price not reached](https://github.com/code-423n4/2022-01-trader-joe-findings/issues/193) 

# Handle

cmichel


# Vulnerability details

In `LaunchEvent.createPair`, when the floor price is not reached (`floorPrice > wavaxReserve * 1e18 / tokenAllocated`), the tokens to be sent to the pool are lowered to match the raised WAVAX at the floor price.

Note that the `floorPrice` is supposed to have a precision of 18:

> /// @param _floorPrice Price of each token in AVAX, scaled to 1e18

The `floorPrice > (wavaxReserve * 1e18) / tokenAllocated` check is correct but the `tokenAllocated` computation involves the `token` decimals:

```solidity
// @audit should be wavaxReserve * 1e18 / floorPrice
tokenAllocated = (wavaxReserve * 10**token.decimals()) / floorPrice;
```

This computation does not work for `token`s that don't have 18 decimals.

#### Example
Assume I want to sell `1.0 wBTC = 1e8 wBTC` (8 decimals) at `2,000.0 AVAX = 2,000 * 1e18 AVAX`.
The `floorPrice` is `2000e18 * 1e18 / 1e8 = 2e31`

Assume the Launch event only raised `1,000.0 AVAX` - half of the floor price for the issued token amount of `1.0 WBTC` (it should therefore allocate only half a WBTC) - and the token amount will be reduced as: `floorPrice = 2e31 > 1000e18 * 1e18 / 1e8 = 1e31 = actualPrice`.
Then, `tokenAllocated = 1000e18 * 1e8 / 2e31 = 1e29 / 2e31 = 0` and no tokens would be allocated, instead of `0.5 WBTC = 0.5e8 WBTC`.

The computation should be `tokenAllocated = wavaxReserve * 1e18 / floorPrice = 1000e18 * 1e18 / 2e31 = 1e39 / 2e31 = 10e38 / 2e31 = 5e7 = 0.5e8`.

#### Recommendation
The new `tokenAllocated` computation should be `tokenAllocated = wavaxReserve * 1e18 / floorPrice;`.


