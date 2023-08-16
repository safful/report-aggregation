## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [[WP-M10] Wrong formula of `getSharesForAmount()` can potentially cause fund loss when being used to calculate the `shares` to be used in `withdraw()`](https://github.com/code-423n4/2022-03-prepo-findings/issues/81) 

# Lines of code

https://github.com/code-423n4/2022-03-prepo/blob/f63584133a0329781609e3f14c3004c1ca293e71/contracts/core/Collateral.sol#L306-L329


# Vulnerability details

In `Collateral`, the getter functions `getAmountForShares()` and `getSharesForAmount()` is using `totalAssets()` instead of `_strategyController.totalValue()`, making the results can be different than the actual shares amount needed to `withdraw()` a certain amount of `_baseToken` and the amount of shares expected to get by `deposit()` a certain amount.

Specifically, `totalAssets()` includes the extra amount of `_baseToken.balanceOf(Collateral)`.

https://github.com/code-423n4/2022-03-prepo/blob/f63584133a0329781609e3f14c3004c1ca293e71/contracts/core/Collateral.sol#L306-L329

```solidity
function getAmountForShares(uint256 _shares)
    external
    view
    override
    returns (uint256)
{
    if (totalSupply() == 0) {
        return _shares;
    }
    return (_shares * totalAssets()) / totalSupply();
}

function getSharesForAmount(uint256 _amount)
    external
    view
    override
    returns (uint256)
{
    uint256 _totalAssets = totalAssets();
    return
        (_totalAssets > 0)
            ? ((_amount * totalSupply()) / _totalAssets)
            : 0;
}
```

https://github.com/code-423n4/2022-03-prepo/blob/f63584133a0329781609e3f14c3004c1ca293e71/contracts/core/Collateral.sol#L339-L343

```solidity
function totalAssets() public view override returns (uint256) {
    return
        _baseToken.balanceOf(address(this)) +
        _strategyController.totalValue();
}
```

https://github.com/code-423n4/2022-03-prepo/blob/f63584133a0329781609e3f14c3004c1ca293e71/contracts/core/Collateral.sol#L137-L148

```solidity
function withdraw(uint256 _amount)
    external
    override
    nonReentrant
    returns (uint256)
{
    require(_withdrawalsAllowed, "Withdrawals not allowed");
    if (_delayedWithdrawalExpiry != 0) {
        _processDelayedWithdrawal(msg.sender, _amount);
    }
    uint256 _owed = (_strategyController.totalValue() * _amount) /
        totalSupply();
    ...
```

### PoC

Given:

- `_baseToken.balanceOf(Collateral)` == 90
- `_strategyController.totalValue()` == 110
- totalSupply of shares = 100

`totalAssets()` returns: 200

`getSharesForAmount(100)` returns: 50, while `withdraw(50)` will actual only get: 55.

When `Collateral` is used by another contract that manages many users' funds, and if it's using `getSharesForAmount()` to calculate the amount of shares needed for a certain amount of underlying tokens to be withdrawn.

This issue can potentially cause fund loss to the user of that contract because it will actually send a lesser amount of `_baseToken` than expected.

### Recommendation

Consider changing `Collateral.totalValue()` to: 

```solidity
function totalAssets() public view override returns (uint256) {
    return
        _strategyController.totalValue();
}
```

