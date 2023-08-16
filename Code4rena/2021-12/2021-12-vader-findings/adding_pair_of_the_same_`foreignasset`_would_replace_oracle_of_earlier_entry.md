## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- LiquidityBasedTWAP

# [Adding pair of the same `foreignAsset` would replace oracle of earlier entry](https://github.com/code-423n4/2021-12-vader-findings/issues/160) 

# Handle

gzeon


# Vulnerability details

## Impact
Oracles are mapped to the `foreignAsset` but not to the specific pair. Pairs with the same `foreignAsset` (e.g. UniswapV2 and Sushi) will be forced to use the same oracle. Generally this should be the expected behavior but there are also possibility that while adding a new pair changed the oracle of an older pair unexpectedly.

## Proof of Concept
https://github.com/code-423n4/2021-12-vader/blob/9fb7f206eaff1863aeeb8f997e0f21ea74e78b49/contracts/lbt/LiquidityBasedTWAP.sol#L271
```
        oracles[foreignAsset] = oracle;
```

## Recommended Mitigation Steps
Bind the oracle to pair instead

