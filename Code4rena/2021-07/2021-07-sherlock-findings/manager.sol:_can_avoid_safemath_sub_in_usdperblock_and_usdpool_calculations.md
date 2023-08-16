## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Manager.sol: Can avoid safemath sub in usdPerBlock and usdPool calculations](https://github.com/code-423n4/2021-07-sherlock-findings/issues/39) 

# Handle

hickuphh3


# Vulnerability details

### Impact

In `updateData()`, 

```jsx
if (sub > add) {
  usdPerBlock = usdPerBlock.sub(sub.sub(add).div(10**18));
} else {
  usdPerBlock = usdPerBlock.add(add.sub(sub).div(10**18));
}
```

we can calculate the difference between `sub` and `add` in both cases without SafeMath because we already know one is greater than the other. 

Also, the logic can be made similar to the `usdPool` calculation since nothing changes in the case where `sub == add`.

The same safemath subtraction avoidance can be implemented for the `usdPool` calculation.

Finally, the variables `sub` and `add` are confusing (makes the code difficult to read because of safemath's add and sub). It is suggested to rename them to `oldUsdPerBlock` and `newUsdPerBlock` respectively.

### Recommended Mitigation Steps

```jsx
// If oldUsdPerBlock == newUsdPerBlock, nothing changes
if (oldUsdPerBlock > newUsdPerBlock) {
  usdPerBlock = usdPerBlock.sub((oldUsdPerBlock - newUsdPerBlock).div(10**18));
} else if (oldUsdPerBlock < newUsdPerBlock) {
  usdPerBlock = usdPerBlock.add((newUsdPerBlock - oldUsdPerBlock).div(10**18));
}

if (_newUsd > _oldUsd) {
	usdPool = usdPool.add((_newUsd - _oldUsd).mul(ps.sherXUnderlying).div(10**18));
} else if (_newUsd < _oldUsd) {
	usdPool = usdPool.sub((_oldUsd - _newUsd).mul(ps.sherXUnderlying).div(10**18));
}
```

