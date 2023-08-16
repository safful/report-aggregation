## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [NFTXMarketplaceZap: Restrict native ETH transfers to WETH contract](https://github.com/code-423n4/2021-12-nftx-findings/issues/224) 

# Handle

GreyArt


# Vulnerability details

## Impact

Native fund transfers into the zap contract are only expected from the WETH contract. Hence, it would be good to restrict incoming fund transfers to prevent accidental native fund transfers from other sources.

This is also true even though `sushiRouter.swapExactTokensForETH()` is called, as the recipient of the swap is expected to not be the marketplace zap contract.

## Recommended Mitigation Steps

Modify the `receive()` function to only accept transfers from the wrapped token contract.

```jsx
receive() external payable {
  require(msg.sender == address(WETH), "Only WETH");
}
```

