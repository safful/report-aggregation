## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [ConstantProductPool: Unnecessary mod before casting to uint32](https://github.com/code-423n4/2021-09-sushitrident-findings/issues/37) 

# Handle

GreyArt


# Vulnerability details

## Impact

Gas optimisation. Casting something of uint256 to uint32 has the same effect as mod since it will wrap around when it overflows. You need to ensure that unchecked math is used otherwise it will revert.

## Recommended Mitigation Steps

[Line 294 of ConstantProductPool.sol](https://github.com/sushiswap/trident/blob/9130b10efaf9c653d74dc7a65bde788ec4b354b5/contracts/pool/ConstantProductPool.sol#L294)

```jsx
function _update(
        uint256 balance0,
        uint256 balance1,
        uint112 _reserve0,
        uint112 _reserve1,
        uint32 _blockTimestampLast
    ) internal {
        require(balance0 <= type(uint112).max && balance1 <= type(uint112).max, "OVERFLOW");
        if (blockTimestampLast == 0) {
            // @dev TWAP support is disabled for gas efficiency.
            reserve0 = uint112(balance0);
            reserve1 = uint112(balance1);
        } else {
						unchecked { // changes starts here
							uint32 blockTimestamp = uint32(block.timestamp);
						} // changes end here
            ...
        }
        emit Sync(balance0, balance1);
    }
```

