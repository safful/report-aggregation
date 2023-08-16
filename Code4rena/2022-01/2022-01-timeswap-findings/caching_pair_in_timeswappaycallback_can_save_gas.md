## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Caching pair in timeswapPayCallback can save gas](https://github.com/code-423n4/2022-01-timeswap-findings/issues/106) 

# Handle

p4st13r4


# Vulnerability details

## Impact

In `CollateralizedDebt.sol` the `pair` state variable is read twice. It can just be immediately assigned locally so that the `require` and the `collateralizedDebtCallback` do not read the same state variable twice

## Proof of Concept

```jsx
function timeswapPayCallback(uint128 assetIn, bytes calldata data) external override {
    require(msg.sender == address(pair), 'E401');

    convenience.collateralizedDebtCallback(pair, maturity, assetIn, data);
}
```

## Tools Used

Editor

## Recommended Mitigation Steps

Assign `pair` to e.g `localPair`

