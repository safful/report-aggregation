## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Unused local variables](https://github.com/code-423n4/2021-12-nftx-findings/issues/163) 

# Handle

WatchPug


# Vulnerability details

Unused local variables in contracts increase contract size and gas usage at deployment.

Instances include:

https://github.com/code-423n4/2021-12-nftx/blob/194073f750b7e2c9a886ece34b6382b4f1355f36/nftx-protocol-v2/contracts/solidity/NFTXStakingZap.sol#L187-L187

```solidity=187
uint256 xTokensMinted = inventoryStaking.timelockMintFor(vaultId, count*BASE, msg.sender, inventoryLockTime);
```

https://github.com/code-423n4/2021-12-nftx/blob/194073f750b7e2c9a886ece34b6382b4f1355f36/nftx-protocol-v2/contracts/solidity/NFTXStakingZap.sol#L207-L207

```solidity=207
uint256 xTokensMinted = inventoryStaking.timelockMintFor(vaultId, count*BASE, msg.sender, inventoryLockTime);
```

