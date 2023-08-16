## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Avoid unnecessary storage writes can save gas](https://github.com/code-423n4/2021-11-nested-findings/issues/162) 

# Handle

WatchPug


# Vulnerability details

`releaseToken()` has `nonReentrant` modifier, making `releaseTokens()` to set storage `_status` multiple times in the for loop.

https://github.com/code-423n4/2021-11-nested/blob/f646002b692ca5fa3631acfff87dda897541cf41/contracts/FeeSplitter.sol#L116-L129

```solidity=116
function releaseToken(IERC20 _token) public nonReentrant {
    uint256 amount = _releaseToken(_msgSender(), _token);
    _token.safeTransfer(_msgSender(), amount);
    emit PaymentReleased(_msgSender(), address(_token), amount);
}

/// @notice Call releaseToken() for multiple tokens
/// @param _tokens ERC20 tokens to release
function releaseTokens(IERC20[] memory _tokens) external {
    for (uint256 i = 0; i < _tokens.length; i++) {
        releaseToken(_tokens[i]);
    }
}
```

### Recommendation

Change to:

```solidity=116
function releaseToken(IERC20 _token) public nonReentrant {
    _releaseTokenAndTransfer(_token);
}

/// @notice Call releaseToken() for multiple tokens
/// @param _tokens ERC20 tokens to release
function releaseTokens(IERC20[] memory _tokens) external {
    for (uint256 i = 0; i < _tokens.length; i++) {
        _releaseTokenAndTransfer(_tokens[i]);
    }
}

function _releaseTokenAndTransfer(IERC20 _token) private {
    uint256 amount = _releaseToken(_msgSender(), _token);
    _token.safeTransfer(_msgSender(), amount);
    emit PaymentReleased(_msgSender(), address(_token), amount);
}
```

