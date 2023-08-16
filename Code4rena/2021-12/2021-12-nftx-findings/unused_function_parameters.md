## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Unused function parameters](https://github.com/code-423n4/2021-12-nftx-findings/issues/164) 

# Handle

WatchPug


# Vulnerability details

Unused function parameters increase contract size and gas usage at deployment.

https://github.com/code-423n4/2021-12-nftx/blob/194073f750b7e2c9a886ece34b6382b4f1355f36/nftx-protocol-v2/contracts/solidity/NFTXMarketplaceZap.sol#L496-L511

```solidity=496
  function _buyVaultToken(
    address vault, 
    uint256 minTokenOut, 
    uint256 maxWethIn, 
    address[] calldata path
  ) internal returns (uint256[] memory) {
    uint256[] memory amounts = sushiRouter.swapTokensForExactTokens(
      minTokenOut,
      maxWethIn,
      path, 
      address(this),
      block.timestamp
    );

    return amounts;
  }
```

`vault` is unused.

