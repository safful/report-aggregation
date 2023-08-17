## Tags

- bug
- 2 (Med Risk)
- primary issue
- selected for report
- sponsor confirmed
- M-20

# [Trading#initiateMarketOrder allows to open a position with more margin than expected due to _handleOpenFees wrong calculation when a trade is referred](https://github.com/code-423n4/2022-12-tigris-findings/issues/542) 

# Lines of code

https://github.com/code-423n4/2022-12-tigris/blob/588c84b7bb354d20cbca6034544c4faa46e6a80e/contracts/Trading.sol#L178-L179
https://github.com/code-423n4/2022-12-tigris/blob/588c84b7bb354d20cbca6034544c4faa46e6a80e/contracts/Trading.sol#L734-L738


# Vulnerability details

## Description
When ```initiateMarketOrder``` is called, ```_marginAfterFees``` are calculated and then use it to calculate ```_positionSize```

```solidity
uint256 _marginAfterFees = _tradeInfo.margin - _handleOpenFees(_tradeInfo.asset, _tradeInfo.margin*_tradeInfo.leverage/1e18, _trader, _tigAsset, false);
uint256 _positionSize = _marginAfterFees * _tradeInfo.leverage / 1e18;
```

The problem is that ```_handleOpenFees``` does not consider referrer fees when it calculates its output (paidFees), leading to open a position greater than expected.

## Impact
For a referred trade, ```initiateMarketOrder``` always opens a position greater than the one supposed, by allowing to use more margin than the one expected.


## POC
The output of ```_handleOpenFees``` is ```_feePaid```, which is calculated [once](https://github.com/code-423n4/2022-12-tigris/blob/588c84b7bb354d20cbca6034544c4faa46e6a80e/contracts/Trading.sol#L734-L738), and it does not consider referralFees

```solidity
// No refferal fees are considered
_feePaid =
    _positionSize
    * (_fees.burnFees + _fees.botFees) // get total fee%
    / DIVISION_CONSTANT // divide by 100%
    + _daoFeesPaid;
```

Then we can notice that, if the output of ```_handleOpenFees``` did not consider referral fees, neither would _marginAfterFees do 
```solidity
uint256 _marginAfterFees =
    _tradeInfo.margin-
    _handleOpenFees(
        _tradeInfo.asset,
        _tradeInfo.margin*_tradeInfo.leverage/1e18, 
        _trader,
        _tigAsset,
        false);

// @audit Then _positionSize would be greater than what is supposed to be, allowing to create a position greater than expected
uint256 _positionSize = _marginAfterFees * _tradeInfo.leverage / 1e18;
```

## Mitigation steps
Consider referral fees when ```_feePaid``` is calculated in ```_handleOpenFees```

```diff
// In _handleOpenFees function
+   uint256 _refFeesToConsider = _referrer == address(0) ? 0 : _fees.referralFees;
    _feePaid =
        _positionSize
-       * (_fees.burnFees + _fees.botFees) // get total fee%
+       * (_fees.burnFees + _fees.botFees + _refFeesToConsider) // get total fee%
        / DIVISION_CONSTANT // divide by 100%
        + _daoFeesPaid;
```