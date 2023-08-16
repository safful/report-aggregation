## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Better Math in `calculateReturn`](https://github.com/code-423n4/2021-09-swivel-findings/issues/161) 

# Handle

0xsanson


# Vulnerability details

## Impact
`Marketplace.calculateReturn` can be rewritten from:
```js
function calculateReturn(address u, uint256 m, uint256 a) internal returns (uint256) {
  // calculate difference between the cToken exchange rate @ maturity and the current cToken exchange rate
  uint256 yield = ((CErc20(markets[u][m].cTokenAddr).exchangeRateCurrent() * 1e26) / maturityRate[u][m]) - 1e26;
  uint256 interest = (yield * a) / 1e26;

  // calculate the total amount of underlying principle to return
  return a + interest;
}
```

to:
```js
function calculateReturn(address u, uint256 m, uint256 a) internal returns (uint256) {
  uint256 rate = CErc20(markets[u][m].cTokenAddr).exchangeRateCurrent();
  return  a*rate/ maturityRate[u][m];
}
```

Less math operations means less approximations and less gas used.

## Proof of Concept
https://github.com/Swivel-Finance/gost/blob/v2/test/marketplace/MarketPlace.sol#L160-L167

## Tools Used
editor

