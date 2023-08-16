## Tags

- bug
- sponsor confirmed
- 2 (Med Risk)

# [Missing `_token.approve()` to `curvePool` in `setZapConfig`](https://github.com/code-423n4/2021-11-badgerzaps-findings/issues/53) 

# Handle

WatchPug


# Vulnerability details

https://github.com/Badger-Finance/badger-ibbtc-utility-zaps/blob/8d265aacb905d30bd95dcd54505fb26dc1f9b0b6/contracts/SettToRenIbbtcZap.sol#L162-L183

```solidity=162
function setZapConfig(
        uint256 _idx,
        address _sett,
        address _token,
        address _curvePool,
        address _withdrawToken,
        int128 _withdrawTokenIndex
    ) external {
        _onlyGovernance();

        require(_sett != address(0));
        require(_token != address(0));
        require(
            _withdrawToken == address(WBTC) || _withdrawToken == address(RENBTC)
        );

        zapConfigs[_idx].sett = ISett(_sett);
        zapConfigs[_idx].token = IERC20Upgradeable(_token);
        zapConfigs[_idx].curvePool = ICurveFi(_curvePool);
        zapConfigs[_idx].withdrawToken = IERC20Upgradeable(_withdrawToken);
        zapConfigs[_idx].withdrawTokenIndex = _withdrawTokenIndex;
    }
```

In the current implementation, when `curvePool` or `token` got updated, `token` is not approved to `curvePool`, which will malfunction the contract and break minting.

### Recommendation

Change to:

```solidity=162
function setZapConfig(
        uint256 _idx,
        address _sett,
        address _token,
        address _curvePool,
        address _withdrawToken,
        int128 _withdrawTokenIndex
    ) external {
        _onlyGovernance();

        require(_sett != address(0));
        require(_token != address(0));
        require(
            _withdrawToken == address(WBTC) || _withdrawToken == address(RENBTC)
        );

        if (zapConfigs[_idx].curvePool != _curvePool && _curvePool != address(0)) {
            IERC20Upgradeable(_token).safeApprove(
                _curvePool,
                type(uint256).max
            );
        }

        zapConfigs[_idx].sett = ISett(_sett);
        zapConfigs[_idx].token = IERC20Upgradeable(_token);
        zapConfigs[_idx].curvePool = ICurveFi(_curvePool);
        zapConfigs[_idx].withdrawToken = IERC20Upgradeable(_withdrawToken);
        zapConfigs[_idx].withdrawTokenIndex = _withdrawTokenIndex;
    }
```

