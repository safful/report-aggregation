## Tags

- bug
- sponsor confirmed
- 0 (Non-critical)

# [ConstantProductPool: Move minting of MIN_LIQUIDITY after checks](https://github.com/code-423n4/2021-09-sushitrident-findings/issues/35) 

# Handle

GreyArt


# Vulnerability details

### Impact

L102: `_mint(address(0), MINIMUM_LIQUIDITY);` should be shifted after the if / else block to L110 because of further checks done in L106, and L108-L109.

This would help save gas should the checks mentioned fail.

### Recommended Mitigation Steps

```jsx
if (msg.sender == migrator) {
  liquidity = IMigrator(migrator).desiredLiquidity();
  require(liquidity != 0 && liquidity != type(uint256).max, "BAD_DESIRED_LIQUIDITY");
} else {
  require(migrator == address(0), "ONLY_MIGRATOR");
  liquidity = computed - MINIMUM_LIQUIDITY;
}
_mint(address(0), MINIMUM_LIQUIDITY);
```

