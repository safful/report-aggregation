## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Strategy Migration May Leave Tokens in the Old Strategy Impacting Share Calculations](https://github.com/code-423n4/2022-03-prepo-findings/issues/26) 

# Lines of code

https://github.com/code-423n4/2022-03-prepo/blob/main/contracts/core/SingleStrategyController.sol#L51-L72


# Vulnerability details

## Impact

If a strategy does not have sufficient funds to `withdraw()` for the full amount then it is possible that tokens will be left in this yield contract during `migrate()`.

It is common for withdrawal from a strategy to withdraw less than a user's balance. The reason is that these yield protocols may lend the deposited funds to borrowers, if there is less funds in the pool than the withdrawal amount the withdrawal may succeed but only transfer the funds available rather than the full withdrawal amount.

The impact of tokens remaining in the old strategy is that when we call `StrategyController.totalValue()` this will only account for the tokens deposited in the new strategy and not those stuck in the previous strategy. Therefore `totalValue()` is undervalued. 

Thus, when a user calls `Collateral.deposit()` the share calculations `_shares = (_amountToDeposit * totalSupply()) / (_valueBefore);` will be over stated (note: `uint256 _valueBefore = _strategyController.totalValue();`). Hence, the user will receive more shares than they should.

The old tokens may be recovered by calling `migrate()` back to the old strategy. If this is done then `totalValue()` will now include the tokens previously stuck. The recent depositer may now withdraw and will be owed `(_strategyController.totalValue() * _amount) / totalSupply()`. Since `totalValue()` is now includes the previously stuck tokens  `_owed` will be overstated and the user will receive more collateral than they should.

The remaining users who had deposited before `migrate()` will lose tokens proportional to their share of the `totalSupply()`.

## Proof of Concept

https://github.com/code-423n4/2022-03-prepo/blob/main/contracts/core/SingleStrategyController.sol#L51-L72
```
    function migrate(IStrategy _newStrategy)
        external
        override
        onlyOwner
        nonReentrant
    {
        uint256 _oldStrategyBalance;
        IStrategy _oldStrategy = _strategy;
        _strategy = _newStrategy;
        _baseToken.approve(address(_newStrategy), type(uint256).max);
        if (address(_oldStrategy) != address(0)) {
            _baseToken.approve(address(_oldStrategy), 0);
            _oldStrategyBalance = _oldStrategy.totalValue();
            _oldStrategy.withdraw(address(this), _oldStrategyBalance);
            _newStrategy.deposit(_baseToken.balanceOf(address(this)));
        }
        emit StrategyMigrated(
            address(_oldStrategy),
            address(_newStrategy),
            _oldStrategyBalance
        );
    }
```

## Recommended Mitigation Steps

The recommendation is to ensure that `require(_oldStrategy.totalValue() == 0)` after calling `_oldStrategy.withdraw()`. This ensures that no funds are left in the strategy. Consider the code example below.

```
    function migrate(IStrategy _newStrategy)
        external
        override
        onlyOwner
        nonReentrant
    {
        uint256 _oldStrategyBalance;
        IStrategy _oldStrategy = _strategy;
        _strategy = _newStrategy;
        _baseToken.approve(address(_newStrategy), type(uint256).max);
        if (address(_oldStrategy) != address(0)) {
            _baseToken.approve(address(_oldStrategy), 0);
            _oldStrategyBalance = _oldStrategy.totalValue();
            _oldStrategy.withdraw(address(this), _oldStrategyBalance);
            require(_oldStrategy.totalValue() == 0)
            _newStrategy.deposit(_baseToken.balanceOf(address(this)));
        }
        emit StrategyMigrated(
            address(_oldStrategy),
            address(_newStrategy),
            _oldStrategyBalance
        );
    }
```

