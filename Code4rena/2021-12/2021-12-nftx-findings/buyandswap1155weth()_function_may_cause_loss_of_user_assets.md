## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed

# [buyAndSwap1155WETH() function may cause loss of user assets](https://github.com/code-423n4/2021-12-nftx-findings/issues/2) 

# Handle

cccz


# Vulnerability details

## Impact

In the NFTXMarketplaceZap.sol contract, the buyAndSwap1155WETH function uses the WETH provided by the user to exchange VaultToken, but when executing the _buyVaultToken method, msg.value is used instead of maxWethIn. Since msg.value is 0, the call will fail.

```
function buyAndSwap1155WETH(
  uint256 vaultId,
  uint256[] memory idsIn,
  uint256[] memory amounts,
  uint256[] memory specificIds,
  uint256 maxWethIn,
  address[] calldata path,
  address to
) public payable nonReentrant {
  require(to != address(0));
  require(idsIn.length != 0);
  IERC20Upgradeable(address(WETH)).transferFrom(msg.sender, address(this), maxWethIn);
  uint256 count;
  for (uint256 i = 0; i <idsIn.length; i++) {
      uint256 amount = amounts[i];
      require(amount> 0, "Transferring <1");
      count += amount;
  }
  INFTXVault vault = INFTXVault(nftxFactory.vault(vaultId));
  uint256 redeemFees = (vault.targetSwapFee() * specificIds.length) + (
      vault.randomSwapFee() * (count-specificIds.length)
  );
  uint256[] memory swapAmounts = _buyVaultToken(address(vault), redeemFees, msg.value, path);
```

In extreme cases, when the user provides both ETH and WETH (the user approves the contract WETH in advance and calls the buyAndSwap1155WETH function instead of the buyAndSwap1155 function by mistake), the _buyVaultToken function will execute successfully, but because the buyAndSwap1155WETH function will not convert ETH to WETH, The user’s ETH will be locked in the contract, causing loss of user assets.

```
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


## Tools Used

Manual audit

## Recommended Mitigation Steps


```
- uint256[] memory swapAmounts = _buyVaultToken(address(vault), redeemFees, msg.value, path);
+ uint256[] memory swapAmounts = _buyVaultToken(address(vault), redeemFees, maxWethIn, path);
```

