## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Code Style: private/internal function names should be prefixed with `_`](https://github.com/code-423n4/2021-11-malt-findings/issues/265) 

# Handle

WatchPug


# Vulnerability details

Here are some examples that the code style of function names does not follow the best practices:

- MaltDAO#incrementEpoch()

    https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/DAO.sol#L106-L119

```solidity=106{107}
/* Internal methods */
function incrementEpoch() internal {
    epoch = epoch.add(1);
}

function _setEpochLength(uint256 length) internal {
    epochLength = length;
    emit SetEpochLength(length);
}

function _setMaltToken(address _malt) internal {
    malt = IBurnMintableERC20(_malt);
    emit SetMaltToken(_malt);
}
```

