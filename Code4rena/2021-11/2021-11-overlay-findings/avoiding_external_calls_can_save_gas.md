## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Avoiding external calls can save gas](https://github.com/code-423n4/2021-11-overlay-findings/issues/74) 

# Handle

WatchPug


# Vulnerability details

Every call to an external contract costs a decent amount of gas. In `OverlayV1OVLCollateral.sol`, `mothership.fee()` can be cached as a storage variable and save ~21000 gas each time.

https://github.com/code-423n4/2021-11-overlay/blob/1833b792caf3eb8756b1ba5f50f9c2ce085e54d0/contracts/collateral/OverlayV1OVLCollateral.sol#L305-L305

```solidity=305
uint _feeAmount = _userNotional.mulUp(mothership.fee());
```

## Recommendation

- Add a storage variable in `OverlayV1OVLCollateral.sol`;
- Add a function `updateFee()`
- Call `updateFee()` after `OverlayV1Mothership.sol#adjustGlobalParams()`

