## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Wrong returns of `SavingsAccountUtil.depositFromSavingsAccount()` can cause fund loss](https://github.com/code-423n4/2021-12-sublime-findings/issues/132) 

# Handle

WatchPug


# Vulnerability details

The function `SavingsAccountUtil.depositFromSavingsAccount()` is expected to return the number of equivalent shares for given `_asset`.

https://github.com/code-423n4/2021-12-sublime/blob/9df1b7c4247f8631647c7627a8da9bdc16db8b11/contracts/Pool/Pool.sol#L225-L267

```solidity
/**
 * @notice internal function used to get amount of collateral deposited to the pool
 * @param _fromSavingsAccount if true, collateral is transferred from _sender's savings account, if false, it is transferred from _sender's wallet
 * @param _toSavingsAccount if true, collateral is transferred to pool's savings account, if false, it is withdrawn from _sender's savings account
 * @param _asset address of the asset to be deposited
 * @param _amount amount of tokens to be deposited in the pool
 * @param _poolSavingsStrategy address of the saving strategy used for collateral deposit
 * @param _depositFrom address which makes the deposit
 * @param _depositTo address to which the tokens are deposited
 * @return _sharesReceived number of equivalent shares for given _asset
 */
function _deposit(
    bool _fromSavingsAccount,
    bool _toSavingsAccount,
    address _asset,
    uint256 _amount,
    address _poolSavingsStrategy,
    address _depositFrom,
    address _depositTo
) internal returns (uint256 _sharesReceived) {
    if (_fromSavingsAccount) {
        _sharesReceived = SavingsAccountUtil.depositFromSavingsAccount(
            ISavingsAccount(IPoolFactory(poolFactory).savingsAccount()),
            _depositFrom,
            _depositTo,
            _amount,
            _asset,
            _poolSavingsStrategy,
            true,
            _toSavingsAccount
        );
    } else {
        _sharesReceived = SavingsAccountUtil.directDeposit(
            ISavingsAccount(IPoolFactory(poolFactory).savingsAccount()),
            _depositFrom,
            _depositTo,
            _amount,
            _asset,
            _toSavingsAccount,
            _poolSavingsStrategy
        );
    }
}
```

However, since `savingsAccountTransfer()` does not return the result of `_savingsAccount.transfer()`, but returned `_amount` instead, which means that `SavingsAccountUtil.depositFromSavingsAccount()` may not return the actual shares (when pps is not 1).


https://github.com/code-423n4/2021-12-sublime/blob/9df1b7c4247f8631647c7627a8da9bdc16db8b11/contracts/SavingsAccount/SavingsAccountUtil.sol#L11-L26

```solidity
    function depositFromSavingsAccount(
        ISavingsAccount _savingsAccount,
        address _from,
        address _to,
        uint256 _amount,
        address _token,
        address _strategy,
        bool _withdrawShares,
        bool _toSavingsAccount
    ) internal returns (uint256) {
        if (_toSavingsAccount) {
            return savingsAccountTransfer(_savingsAccount, _from, _to, _amount, _token, _strategy);
        } else {
            return withdrawFromSavingsAccount(_savingsAccount, _from, _to, _amount, _token, _strategy, _withdrawShares);
        }
    }
```

https://github.com/code-423n4/2021-12-sublime/blob/9df1b7c4247f8631647c7627a8da9bdc16db8b11/contracts/SavingsAccount/SavingsAccountUtil.sol#L66-L80

```solidity
    function savingsAccountTransfer(
        ISavingsAccount _savingsAccount,
        address _from,
        address _to,
        uint256 _amount,
        address _token,
        address _strategy
    ) internal returns (uint256) {
        if (_from == address(this)) {
            _savingsAccount.transfer(_amount, _token, _strategy, _to);
        } else {
            _savingsAccount.transferFrom(_amount, _token, _strategy, _from, _to);
        }
        return _amount;
    }
```

As a result, the recorded `_sharesReceived` can be wrong.


https://github.com/code-423n4/2021-12-sublime/blob/9df1b7c4247f8631647c7627a8da9bdc16db8b11/contracts/Pool/Pool.sol#L207-L223

```solidity
    function _depositCollateral(
        address _depositor,
        uint256 _amount,
        bool _transferFromSavingsAccount
    ) internal nonReentrant {
        uint256 _sharesReceived = _deposit(
            _transferFromSavingsAccount,
            true,
            poolConstants.collateralAsset,
            _amount,
            poolConstants.poolSavingsStrategy,
            _depositor,
            address(this)
        );
        poolVariables.baseLiquidityShares = poolVariables.baseLiquidityShares.add(_sharesReceived);
        emit CollateralAdded(_depositor, _amount, _sharesReceived);
    }
```

### PoC

Given:

- the price per share of yearn USDC vault is `1.2`

1. Alice deposited `12,000 USDC` to `yearn` strategy, received `10,000` share tokens;
2. Alice created a pool, and added all the `12,000 USDC` from the saving account as collateral; The recorded `CollateralAdded` got the wrong number: `12000` which should be `10000`;
3. Alice failed to borrow money with the pool and tries to `cancelPool()`, it fails as the recorded collateral `shares` are more than the actual collateral.

As a result, Alice has lost all the `12,000 USDC`.

If Alice managed to borrow with the pool, when the loan defaults, the liquidation will also fail, and cause fund loss to the lenders.

### Recommendation

Change to:

```solidity
function savingsAccountTransfer(
    ISavingsAccount _savingsAccount,
    address _from,
    address _to,
    uint256 _amount,
    address _token,
    address _strategy
) internal returns (uint256) {
    if (_from == address(this)) {
        return _savingsAccount.transfer(_amount, _token, _strategy, _to);
    } else {
        return _savingsAccount.transferFrom(_amount, _token, _strategy, _from, _to);
    }
}
```

