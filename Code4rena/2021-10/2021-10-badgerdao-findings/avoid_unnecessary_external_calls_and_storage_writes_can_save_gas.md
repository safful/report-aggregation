## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Avoid unnecessary external calls and storage writes can save gas](https://github.com/code-423n4/2021-10-badgerdao-findings/issues/58) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-10-badgerdao/blob/9c0ea7b3b02675211446f6c81750c5f3c0a86370/contracts/WrappedIbbtcEth.sol#L69-L77

```solidity
/// @dev Update live ibBTC price per share from core
/// @dev We cache this to reduce gas costs of mint / burn / transfer operations.
/// @dev Update function is permissionless, and must be updated at least once every X time as a sanity check to ensure value is up-to-date
function updatePricePerShare() public virtual returns (uint256) {
    pricePerShare = core.pricePerShare();
    lastPricePerShareUpdate = now;

    emit SetPricePerShare(pricePerShare, lastPricePerShareUpdate);
}
```

Per the comment above `function updatePricePerShare()`, `updatePricePerShare()` may get called quite often when `wibBTC` token is being used more often.

There could potentially be multiple calls to `updatePricePerShare()` in one block. In that case, checking if `pricePerShare` was updated earlier in the same block can save some gas from unnecessary external calls and storage writes.

### Recommendation

Change to:

```solidity
function updatePricePerShare() public virtual returns (uint256) {
    if (lastPricePerShareUpdate < now) {
        pricePerShare = core.pricePerShare();
        lastPricePerShareUpdate = now;

        emit SetPricePerShare(pricePerShare, lastPricePerShareUpdate);
    }
}
```

