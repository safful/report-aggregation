## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Unused events](https://github.com/code-423n4/2021-12-nftx-findings/issues/162) 

# Handle

WatchPug


# Vulnerability details

Unused events increase contract size and gas usage at deployment.

https://github.com/code-423n4/2021-12-nftx/blob/194073f750b7e2c9a886ece34b6382b4f1355f36/nftx-protocol-v2/contracts/solidity/eligibility/NFTXMintRequestEligibility.sol#L62-L62

```solidity
event Reject(uint256[] nftIds);
```

`Reject` is unused.

