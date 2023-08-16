## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed
- Oracles

# [CompositeMultiOracle.sol - bases.length in setSources() and setPaths() can be stored in a variable](https://github.com/code-423n4/2021-08-yield-findings/issues/16) 

# Handle

PierrickGT


# Vulnerability details

## Impact
In the external functions `setSources` and `setPaths` of `CompositeMultiOracle.sol`, we call `bases.length` three times. This value can be stored in a variable to avoid two sload.

Also, `uint256 i = 0;` can be refactored to `uint256 i;` since `uint256` initializes with a value of `0` by default.

## Proof of Concept
https://github.com/code-423n4/2021-08-yield/blob/e4227756bd2b74d683525d61f0636bed64325955/contracts/oracles/composite/CompositeMultiOracle.sol#L39-L46

https://github.com/code-423n4/2021-08-yield/blob/e4227756bd2b74d683525d61f0636bed64325955/contracts/oracles/composite/CompositeMultiOracle.sol#L60-L67

## Recommended Mitigation Steps
Store `bases.length` in a variable and declare `i` with the default value:
```
uint256 basesLength = bases.length;

require(
    basesLength == quotes.length && 
    basesLength == sources_.length,
    "Mismatched inputs"
);

for (uint256 i; i < basesLength; i++) {
    _setSource(bases[i], quotes[i], sources_[i]);
}
```
```
uint256 basesLength = bases.length;

require(
    basesLength == quotes.length && 
    basesLength == paths_.length,
    "Mismatched inputs"
);

for (uint256 i; i < basesLength; i++) {
    _setPath(bases[i], quotes[i], paths_[i]);
}
```

