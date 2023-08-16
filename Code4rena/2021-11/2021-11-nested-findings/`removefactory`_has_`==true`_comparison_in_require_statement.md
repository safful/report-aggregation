## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [`removeFactory` has `==true` comparison in require statement](https://github.com/code-423n4/2021-11-nested-findings/issues/63) 

# Handle

loop


# Vulnerability details

The `removeFactory` has an unnecessary `==true` comparison in its require statement. Since require already checks if the condition is `true`, there is no need for it to be compared.

## Impact
Removing `== true` saves a tiny amount of gas if `removeFactory` is called.

## Proof of Concept
https://github.com/code-423n4/2021-11-nested/blob/main/contracts/NestedAsset.sol#L142

