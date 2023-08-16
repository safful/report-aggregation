## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [`10**30` can be changed to `1e30` and save some gas](https://github.com/code-423n4/2021-12-sublime-findings/issues/126) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-12-sublime/blob/9df1b7c4247f8631647c7627a8da9bdc16db8b11/contracts/CreditLine/CreditLine.sol#L379-L383

```solidity
    function _updateLiquidatorRewardFraction(uint256 _rewardFraction) internal {
        require(_rewardFraction <= 10**30, 'Fraction has to be less than 1');
        liquidatorRewardFraction = _rewardFraction;
        emit LiquidationRewardFractionUpdated(_rewardFraction);
    }
```

Can be changed to:

```solidity
    function _updateLiquidatorRewardFraction(uint256 _rewardFraction) internal {
        require(_rewardFraction <= 1e30, 'Fraction has to be less than 1');
        liquidatorRewardFraction = _rewardFraction;
        emit LiquidationRewardFractionUpdated(_rewardFraction);
    }
```

and save some gas from unnecessary arithmetic operation in `10**30`.

