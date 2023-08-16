## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Cache external call result in the stack can save gas](https://github.com/code-423n4/2021-10-badgerdao-findings/issues/51) 

# Handle

WatchPug


# Vulnerability details

For the result of an external call being written into a storage variable, cache and read from the stack rather than read from the storage variable can save gas.

https://github.com/code-423n4/2021-10-badgerdao/blob/9c0ea7b3b02675211446f6c81750c5f3c0a86370/contracts/WrappedIbbtcEth.sol#L72-L77

```solidity
function updatePricePerShare() public virtual returns (uint256) {
    pricePerShare = core.pricePerShare();
    lastPricePerShareUpdate = now;

    emit SetPricePerShare(pricePerShare, lastPricePerShareUpdate);
}
```

### Recommendation

Change to:

```solidity
function updatePricePerShare() public virtual returns (uint256) {
    uint256 lastPricePerShare = core.pricePerShare();
    pricePerShare = lastPricePerShare;
    lastPricePerShareUpdate = now;

    emit SetPricePerShare(lastPricePerShare, lastPricePerShareUpdate);
}
```

