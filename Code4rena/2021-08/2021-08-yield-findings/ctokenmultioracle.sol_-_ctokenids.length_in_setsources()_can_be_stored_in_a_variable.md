## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed
- Oracles

# [CTokenMultiOracle.sol - cTokenIds.length in setSources() can be stored in a variable](https://github.com/code-423n4/2021-08-yield-findings/issues/13) 

# Handle

PierrickGT


# Vulnerability details

## Impact
In the external function `setSources` of `CTokenMultiOracle.sol`, we call `cTokenIds.length` three times. This value can be stored in a variable to avoid two sload.

Also, `uint256 i = 0;` can be refactored to `uint256 i;` since `uint256` initializes with a value of `0` by default.

## Proof of Concept
https://github.com/code-423n4/2021-08-yield/blob/1383f6a715657547603cddd0fed824cde631c7db/contracts/oracles/compound/CTokenMultiOracle.sol#L38-L39

https://github.com/code-423n4/2021-08-yield/blob/1383f6a715657547603cddd0fed824cde631c7db/contracts/oracles/compound/CTokenMultiOracle.sol#L42

## Recommended Mitigation Steps
Store `cTokenIds.length` in a variable and declare `i` with the default value:
```
uint256 cTokenIdsLength = cTokenIds.length;

require(
    cTokenIdsLength == underlyings.length &&
    cTokenIdsLength == cTokens.length,
    "Mismatched inputs"
);

for (uint256 i; i < cTokenIdsLength; i++) {
    _setSource(cTokenIds[i], underlyings[i], cTokens[i]);
}
```

