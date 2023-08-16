## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [`Alchemist.sol#mint()` Two storage writes can be combined into one](https://github.com/code-423n4/2021-11-yaxis-findings/issues/53) 

# Handle

WatchPug


# Vulnerability details

In `Alchemist.sol#mint()`, when `borrowFee > 0`, `_cdp.totalDebt` will be written 2 times. Combing them into one storage write can save gas.

https://github.com/code-423n4/2021-11-yaxis/blob/146febcb61ae7fe20b0920849c4f4bbe111c6ba7/contracts/v3/alchemix/Alchemist.sol#L611-L645

```solidity
function mint(uint256 _amount)
    external
    nonReentrant
    noContractAllowed
    onPriceCheck
    expectInitialized
{
    CDP.Data storage _cdp = _cdps[msg.sender];
    _cdp.update(_ctx);

    uint256 _totalCredit = _cdp.totalCredit;

    if (_totalCredit < _amount) {
        uint256 _remainingAmount = _amount.sub(_totalCredit);

        if (borrowFee > 0) {
            uint256 _borrowFeeAmount = _remainingAmount.mul(borrowFee).div(
                PERCENT_RESOLUTION
            );
            _cdp.totalDebt = _cdp.totalDebt.add(_borrowFeeAmount);
            xtoken.mint(rewards, _borrowFeeAmount);
        }
        _cdp.totalDebt = _cdp.totalDebt.add(_remainingAmount);
        _cdp.totalCredit = 0;

        _cdp.checkHealth(_ctx, 'Alchemist: Loan-to-value ratio breached');
    } else {
        _cdp.totalCredit = _totalCredit.sub(_amount);
    }

    xtoken.mint(msg.sender, _amount);
    if (_amount >= flushActivator) {
        flushActiveVault();
    }
}
```

### Recommendation

Change to:

```solidity
function mint(uint256 _amount)
    external
    nonReentrant
    noContractAllowed
    onPriceCheck
    expectInitialized
{
    CDP.Data storage _cdp = _cdps[msg.sender];
    _cdp.update(_ctx);

    uint256 _totalCredit = _cdp.totalCredit;
    uint256 _totalDebt = _cdp.totalDebt;

    if (_totalCredit < _amount) {
        uint256 _remainingAmount = _amount.sub(_totalCredit);

        if (borrowFee > 0) {
            uint256 _borrowFeeAmount = _remainingAmount.mul(borrowFee).div(
                PERCENT_RESOLUTION
            );
            _totalDebt = _totalDebt.add(_borrowFeeAmount);
            xtoken.mint(rewards, _borrowFeeAmount);
        }
        _cdp.totalDebt = _totalDebt.add(_remainingAmount);
        _cdp.totalCredit = 0;

        _cdp.checkHealth(_ctx, 'Alchemist: Loan-to-value ratio breached');
    } else {
        _cdp.totalCredit = _totalCredit.sub(_amount);
    }

    xtoken.mint(msg.sender, _amount);
    if (_amount >= flushActivator) {
        flushActiveVault();
    }
}
```

