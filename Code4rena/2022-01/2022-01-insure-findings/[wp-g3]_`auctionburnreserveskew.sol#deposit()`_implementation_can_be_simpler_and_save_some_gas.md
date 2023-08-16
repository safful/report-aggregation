## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [[WP-G3] `AuctionBurnReserveSkew.sol#deposit()` Implementation can be simpler and save some gas](https://github.com/code-423n4/2022-01-insure-findings/issues/215) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/CDSTemplate.sol#L140-L148

```solidity
if (_supply > 0 && _liquidity > 0) {
    _mintAmount = (_amount * _supply) / _liquidity;
} else if (_supply > 0 && _liquidity == 0) {
    //when vault lose all underwritten asset = 
    _mintAmount = _amount * _supply; //dilute LP token value af. Start CDS again.
} else {
    //when _supply == 0,
    _mintAmount = _amount;
}
```

### Recommendation

Change to:

```solidity
if (_supply == 0) {
    _mintAmount = _amount;
} else {
    _mintAmount = _liquidity == 0 ? _amount * _supply : (_amount * _supply) / _liquidity;
} 
```

- Removed 2 checks;
- Removed 1 branch;
- Simpler branch (costs less gas) goes first.

