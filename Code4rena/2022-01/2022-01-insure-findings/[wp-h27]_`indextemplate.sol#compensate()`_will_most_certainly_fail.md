## Tags

- bug
- 3 (High Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [[WP-H27] `IndexTemplate.sol#compensate()` will most certainly fail](https://github.com/code-423n4/2022-01-insure-findings/issues/269) 

# Handle

WatchPug


# Vulnerability details

## Root Cause

Precision loss while converting between `the amount of shares` and `the amount of underlying tokens` back and forth is not handled properly.

---

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/IndexTemplate.sol#L438-L447

```solidity
uint256 _shortage;
if (totalLiquidity() < _amount) {
    //Insolvency case
    _shortage = _amount - _value;
    uint256 _cds = ICDSTemplate(registry.getCDS(address(this)))
        .compensate(_shortage);
    _compensated = _value + _cds;
}
vault.offsetDebt(_compensated, msg.sender);
```

In the current implementation, when someone tries to resume the market after a pending period ends by calling `PoolTemplate.sol#resume()`, `IndexTemplate.sol#compensate()` will be called internally to make a payout. If the index pool is unable to cover the compensation, the CDS pool will then be used to cover the shortage.

However, while `CDSTemplate.sol#compensate()` takes a parameter for the amount of underlying tokens, it uses `vault.transferValue()` to transfer corresponding `_attributions` (shares) instead of underlying tokens.

Due to precision loss, the `_attributions` transferred in the terms of underlying tokens will most certainly be less than the shortage.

At L444, the contract believes that it's been compensated for `_value + _cds`, which is lower than the actual value, due to precision loss.

At L446, when it calls `vault.offsetDebt(_compensated, msg.sender)`, the tx will revert at `require(underlyingValue(msg.sender) >= _amount)`.

As a result, `resume()` can not be done, and the debt can't be repaid.

### PoC 

Given:

- vault.underlyingValue = 10,000
- vault.valueAll = 30,000
- totalAttributions = 2,000,000
- _amount = 1,010,000

0. _shortage = _amount - vault.underlyingValue = 1,000,000
1. _attributions = (_amount * totalAttributions) / valueAll = 67,333,333
2. actualValueTransfered = (valueAll * _attributions) / totalAttributions = 1009999

**Expected results**: actualValueTransfered = _shortage;

**Actual results**: actualValueTransfered < _shortage.

## Impact

The precision loss isn't just happening on special numbers, but will most certainly always revert the txs.

This will malfunction the contract as the index pool can not `compensate()`, therefore the pool can not `resume()`. Causing the funds of the LPs of the pool and the index pool to be frozen, and other stakeholders of the same vault will suffer fund loss from an unfair share of the funds compensated before.

## Recommendation

Change to:

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/IndexTemplate.sol#L439-L446

```solidity
if (totalLiquidity() < _amount) {
    //Insolvency case
    _shortage = _amount - _value;
    uint256 _cds = ICDSTemplate(registry.getCDS(address(this)))
        .compensate(_shortage);
    _compensated = vault.underlyingValue(address(this));
}
vault.offsetDebt(_compensated, msg.sender);
```

