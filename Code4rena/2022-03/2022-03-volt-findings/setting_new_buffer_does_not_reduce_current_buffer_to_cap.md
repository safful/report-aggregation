## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Setting new buffer does not reduce current buffer to cap](https://github.com/code-423n4/2022-03-volt-findings/issues/29) 

# Lines of code

https://github.com/code-423n4/2022-03-volt/blob/f1210bf3151095e4d371c9e9d7682d9031860bbd/contracts/utils/RateLimited.sol#L142


# Vulnerability details

## Impact
The `RateLimited.setBufferCap` function first updates the buffer and then sets the new cap, but does not apply the new cap to the updated buffer.
Meaning, the updated buffer value can be larger than the new buffer cap which should never be the case.
Actions consuming more than the new buffer cap can be performed.

```solidity
function _setBufferCap(uint256 newBufferCap) internal {
    // @audit still uses old buffer cap, should set buffer first
    _updateBufferStored();

    uint256 oldBufferCap = bufferCap;
    bufferCap = newBufferCap;


    emit BufferCapUpdate(oldBufferCap, newBufferCap);
}
```

## Recommended Mitigation Steps
Update the buffer after setting the new cap:

```diff
function _setBufferCap(uint256 newBufferCap) internal {
-   _updateBufferStored();
    uint256 oldBufferCap = bufferCap;
    bufferCap = newBufferCap;

+   _updateBufferStored();

    emit BufferCapUpdate(oldBufferCap, newBufferCap);
}
```


