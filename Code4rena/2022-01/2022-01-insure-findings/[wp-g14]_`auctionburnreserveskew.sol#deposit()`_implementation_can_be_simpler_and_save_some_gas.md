## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [[WP-G14] `AuctionBurnReserveSkew.sol#deposit()` Implementation can be simpler and save some gas](https://github.com/code-423n4/2022-01-insure-findings/issues/233) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/CDSTemplate.sol#L260-L270

```solidity
if (_available >= _amount) {
    _compensated = _amount;
    _attributionLoss = vault.transferValue(_amount, msg.sender);
    emit Compensated(msg.sender, _amount);
} else {
    //when CDS cannot afford, pay as much as possible
    _compensated = _available;
    _attributionLoss = vault.transferValue(_available, msg.sender);
    emit Compensated(msg.sender, _available);
}

```

### Recommendation

Change to:

```solidity
_compensated = _available >= _amount? _amount: _available;

_attributionLoss = vault.transferValue(_compensated, msg.sender);
emit Compensated(msg.sender, _compensated);
```

- Duplicated codes removed;
- Shorter and simpler code.

