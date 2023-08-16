## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas saving caching the value](https://github.com/code-423n4/2022-01-insure-findings/issues/118) 

# Handle

0x1f8b


# Vulnerability details

## Impact
Gas saving.

## Proof of Concept
There are multiple methods in `Registry` that check a value inside the storage and if it's not defined, use the default one. It's better to cache the value in order to save gas if it was defined avoiding double reading.

For example, instead of the following code:
```
    function getCDS(address _address) external view override returns (address) {
        if (cds[_address] == address(0)) {
            return cds[address(0)];
        } else {
            return cds[_address];
        }
    }
```
use
```
    function getCDS(address _address) external view override returns (address) {
        address val =cds[_address];
        if ( val== address(0)) {
            return cds[address(0)];
        } else {
            return val;
        }
    }
```

## Tools Used
Manual review.

## Recommended Mitigation Steps
Cache the value.

