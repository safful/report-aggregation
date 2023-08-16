## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [`buyAndSwap1155WETH` Does Not Work As Intended](https://github.com/code-423n4/2021-12-nftx-findings/issues/45) 

# Handle

leastwood


# Vulnerability details

## Impact

The `buyAndSwap1155WETH` function in `NFTXMarketplaceZap` aims to facilitate buying and swapping `ERC1155` tokens within a single transaction. The function expects to transfer `WETH` tokens from the `msg.sender` account and use these tokens in purchasing vault tokens. However, the `_buyVaultToken` call in `buyAndSwap1155WETH` actually uses `msg.value` and not `maxWethIn`. As a result, the function will not work unless the user supplies both `WETH` and native `ETH` amounts, equivalent to the `maxWethIn` amount.

## Proof of Concept

https://github.com/code-423n4/2021-12-nftx/blob/main/nftx-protocol-v2/contracts/solidity/NFTXMarketplaceZap.sol#L284-L314
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
  for (uint256 i = 0; i < idsIn.length; i++) {
      uint256 amount = amounts[i];
      require(amount > 0, "Transferring < 1");
      count += amount;
  }
  INFTXVault vault = INFTXVault(nftxFactory.vault(vaultId));
  uint256 redeemFees = (vault.targetSwapFee() * specificIds.length) + (
      vault.randomSwapFee() * (count - specificIds.length)
  );
  uint256[] memory swapAmounts = _buyVaultToken(address(vault), redeemFees, msg.value, path);
  _swap1155(vaultId, idsIn, amounts, specificIds, to);

  emit Swap(count, swapAmounts[0], to);

  // Return extras.
  uint256 remaining = WETH.balanceOf(address(this));
  WETH.transfer(to, remaining);
}
```

## Tools Used

Manual code review.
Discussions with Kiwi.

## Recommended Mitigation Steps

Consider updating the `buyAndSwap1155WETH` function such that the following line of code is used instead of [this](https://github.com/code-423n4/2021-12-nftx/blob/main/nftx-protocol-v2/contracts/solidity/NFTXMarketplaceZap.sol#L306).

```
uint256[] memory swapAmounts = _buyVaultToken(address(vault), redeemFees, maxWethIn, path);
```

