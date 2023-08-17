## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Updating rate limit for addresses restores their entire buffer amount](https://github.com/code-423n4/2022-03-volt-findings/issues/27) 

# Lines of code

https://github.com/code-423n4/2022-03-volt/blob/f1210bf3151095e4d371c9e9d7682d9031860bbd/contracts/utils/MultiRateLimited.sol#L280


# Vulnerability details

## Impact
When the `bufferCap` is updated for an address in `_updateAddress`, the address's allowed buffer (`bufferStored`) is replenished to the entire `bufferCap`.

The address could frontrun the `updateAddress` call and spend their entire buffer, then the buffer is replenished and they can spend their entire buffer a second time.

## Recommended Mitigation Steps
Keep the old buffer value, capped by the new `bufferCap`:

```diff
+ uint256 newBuffer = individualBuffer(rateLimitedAddress);

  rateLimitData.lastBufferUsedTime = block.timestamp.toUint32();
  rateLimitData.bufferCap = _bufferCap;
  rateLimitData.rateLimitPerSecond = _rateLimitPerSecond;
- rateLimitData.bufferStored = _bufferCap;
+ rateLimitData.bufferStored = min(_bufferCap, newBuffer);
```


