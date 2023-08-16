## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Remove unnecessary variables can make the code simpler and save some gas](https://github.com/code-423n4/2021-12-amun-findings/issues/214) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-12-amun/blob/98f6e2ff91f5fcebc0489f5871183566feaec307/contracts/basket/contracts/singleJoinExit/SingleTokenJoin.sol#L51-L57

```solidity
IERC20 inputToken = IERC20(_joinTokenStruct.inputToken);

inputToken.safeTransferFrom(
    msg.sender,
    address(this),
    _joinTokenStruct.inputAmount
);
```

`inputToken` is unnecessary as it's being used only once. Can be changed to:

```solidity
IERC20(_joinTokenStruct.inputToken).safeTransferFrom(
    msg.sender,
    address(this),
    _joinTokenStruct.inputAmount
);
```

