## Tags

- bug
- 3 (High Risk)
- primary issue
- selected for report
- sponsor confirmed
- upgraded by judge
- H-11

# [Not enough margin pulled or burned from user when adding to a position](https://github.com/code-423n4/2022-12-tigris-findings/issues/659) 

# Lines of code

https://github.com/code-423n4/2022-12-tigris/blob/588c84b7bb354d20cbca6034544c4faa46e6a80e/contracts/Trading.sol#L275-L282


# Vulnerability details

## Impact
When adding to a position, the amount of margin pulled from the user is not as much as it should be, which leaks value from the protocol and lowering the collateralization ratio of `tigAsset`.

## Proof of Concept
In `Trading.addToPosition` the `_handleDeposit` function is called like this:

```js
_handleDeposit(
    _trade.tigAsset,
    _marginAsset,
    _addMargin - _fee,
    _stableVault,
    _permitData,
    _trader
);
```

The third parameter with the value of `_addMargin - _fee` is the amount pulled (or burned in the case of using `tigAsset`) from the user. The `_fee` value is calculated as part of the position size like this: 

```js
uint _fee = _handleOpenFees(_trade.asset, _addMargin*_trade.leverage/1e18, _trader, _trade.tigAsset, false);
```

The `_handleOpenFees` function mints `_tigAsset` to the referrer, to the `msg.sender` (if called by a function meant to be executed by bots) and to the protocol itself. Those minted tokens are supposed to be part of the `_addMargin` value paid by the user. Hence using `_addMargin - _fee` as the third parameter to `_handleDeposit` is going to pull or burn less margin than what was accounted for.

An example for correct usage can be seen in `initiateMarketOrder`:

```js
uint256 _marginAfterFees = _tradeInfo.margin - _handleOpenFees(_tradeInfo.asset, _tradeInfo.margin*_tradeInfo.leverage/1e18, _trader, _tigAsset, false);
uint256 _positionSize = _marginAfterFees * _tradeInfo.leverage / 1e18;
_handleDeposit(_tigAsset, _tradeInfo.marginAsset, _tradeInfo.margin, _tradeInfo.stableVault, _permitData, _trader);
```

Here the third parameter to `_handleDeposit` is not `_marginAfterFees` but `_tradeInfo.margin` which is what the user has input and is supposed to pay.


## Tools Used

Manual Review

## Recommended Mitigation Steps

In `Trading.addToPosition` call the `_handleDeposit` function without subtracting the `_fee` value:

```js
_handleDeposit(
    _trade.tigAsset,
    _marginAsset,
    _addMargin,
    _stableVault,
    _permitData,
    _trader
);
```