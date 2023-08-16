## Tags

- bug
- duplicate
- 0 (Non-critical)
- resolved
- sponsor confirmed

# [Unchecked return value for `ERC20.approve` call](https://github.com/code-423n4/2021-12-nftx-findings/issues/87) 

# Handle

WatchPug


# Vulnerability details

There are many functions across the codebase that will perform an ERC20.approve() call but does not check the success return value. Some tokens do not revert if the approval failed but return false instead.

Instances include:

https://github.com/code-423n4/2021-12-nftx/blob/194073f750b7e2c9a886ece34b6382b4f1355f36/nftx-protocol-v2/contracts/solidity/other/PalmNFTXStakingZap.sol#L167-L167

```solidity=167
IERC20Upgradeable(address(_pairedToken)).approve(_sushiRouter, type(uint256).max);
```

https://github.com/code-423n4/2021-12-nftx/blob/194073f750b7e2c9a886ece34b6382b4f1355f36/nftx-protocol-v2/contracts/solidity/other/PalmNFTXStakingZap.sol#L313

https://github.com/code-423n4/2021-12-nftx/blob/194073f750b7e2c9a886ece34b6382b4f1355f36/nftx-protocol-v2/contracts/solidity/other/PalmNFTXStakingZap.sol#L299

https://github.com/code-423n4/2021-12-nftx/blob/194073f750b7e2c9a886ece34b6382b4f1355f36/nftx-protocol-v2/contracts/solidity/NFTXMarketplaceZap.sol#L519

https://github.com/code-423n4/2021-12-nftx/blob/194073f750b7e2c9a886ece34b6382b4f1355f36/nftx-protocol-v2/contracts/solidity/NFTXMarketplaceZap.sol#L538

https://github.com/code-423n4/2021-12-nftx/blob/194073f750b7e2c9a886ece34b6382b4f1355f36/nftx-protocol-v2/contracts/solidity/NFTXSimpleFeeDistributor.sol#L159

https://github.com/code-423n4/2021-12-nftx/blob/194073f750b7e2c9a886ece34b6382b4f1355f36/nftx-protocol-v2/contracts/solidity/NFTXStakingZap.sol#L398

https://github.com/code-423n4/2021-12-nftx/blob/194073f750b7e2c9a886ece34b6382b4f1355f36/nftx-protocol-v2/contracts/solidity/NFTXStakingZap.sol#L171

It is usually good to add a require-statement that checks the return value or to use something like `safeApprove`; unless one is sure the given token reverts in case of a failure.

