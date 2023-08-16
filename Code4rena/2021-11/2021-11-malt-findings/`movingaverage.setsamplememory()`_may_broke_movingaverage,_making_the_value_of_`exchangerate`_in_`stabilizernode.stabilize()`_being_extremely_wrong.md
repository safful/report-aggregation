## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [`MovingAverage.setSampleMemory()` may broke MovingAverage, making the value of `exchangeRate` in `StabilizerNode.stabilize()` being extremely wrong](https://github.com/code-423n4/2021-11-malt-findings/issues/313) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/MovingAverage.sol#L424-L442

```solidity=424
  function setSampleMemory(uint256 _sampleMemory)
    external
    onlyRole(ADMIN_ROLE, "Must have admin privs")
  {
    require(_sampleMemory > 0, "Cannot have sample memroy of 0");

    if (_sampleMemory > sampleMemory) {
      for (uint i = sampleMemory; i < _sampleMemory; i++) {
        samples.push();
      }
      counter = counter % _sampleMemory;
    } else {
      activeSamples = _sampleMemory;

      // TODO handle when list is smaller Tue 21 Sep 2021 22:29:41 BST
    }

    sampleMemory = _sampleMemory;
  }
```

In the current implementation, when `sampleMemory` is updated, the samples index will be malposition, making `getValueWithLookback()` get the wrong samples, so that returns the wrong value.

## PoC

-  When initial sampleMemory is `10`
-  After `movingAverage.update(1e18)` being called for 120 times
-  The admin calls `movingAverage.setSampleMemory(118)` and set sampleMemory to `118`

The current `movingAverage.getValueWithLookback(sampleLength * 10)` returns `0.00000203312 e18`, while it's expeceted to be `1e18`

After `setSampleMemory()`, `getValueWithLookback()` may also return `0`or revert FullMath: FULLDIV_OVERFLOW at L134.

### Recommendation

Consider removing `setSampleMemory` function.

