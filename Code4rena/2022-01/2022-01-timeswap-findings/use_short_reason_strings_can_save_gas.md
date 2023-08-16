## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Use short reason strings can save gas](https://github.com/code-423n4/2022-01-timeswap-findings/issues/171) 

# Handle

WatchPug


# Vulnerability details

Every reason string takes at least 32 bytes.

Use short reason strings that fits in 32 bytes or it will become more expensive.

Instances include:

https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Convenience/contracts/base/ERC721.sol#L55-L58

```solidity
require(
    owner == msg.sender || isApprovedForAll[owner][msg.sender],
    'ERC721 :: approve : Approve caller is not owner nor approved for all'
);
```

https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Convenience/contracts/base/ERC721.sol#L96-L99

```solidity
require(
    _checkOnERC721Received(address(0), to, id, ''),
    'ERC721 :: _safeMint : Transfer to non ERC721Receiver implementer'
);
```

