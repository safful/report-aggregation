## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [[WP-G6] Remove redundant code can save gas](https://github.com/code-423n4/2022-01-insure-findings/issues/218) 

# Handle

WatchPug


# Vulnerability details

Removing `return 0` can make the code simpler and save some gas.

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/CDSTemplate.sol#L295-L303

```solidity
    function rate() external view returns (uint256) {
        if (totalSupply() > 0) {
            return
                (vault.attributionValue(crowdPool) * MAGIC_SCALE_1E6) /
                totalSupply();
        } else {
            return 0;
        }
    }
```

### Recommendation

Can be changed to:

```solidity
    function rate() external view returns (uint256) {
        if (totalSupply() > 0) {
            return
                (vault.attributionValue(crowdPool) * MAGIC_SCALE_1E6) /
                totalSupply();
        } 
    }
```

Other examples include:

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/CDSTemplate.sol#L312-L317

```solidity
        if (_balance == 0) {
            return 0;
        } else {
            return
                _balance * vault.attributionValue(crowdPool) / totalSupply();
        }
```

Can be changed to:

```solidity
if (_balance > 0) {
    return
        _balance * vault.attributionValue(crowdPool) / totalSupply();
} 
```

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/Factory.sol#L176-L176

```solidity
for (uint256 i = 0; i < _references.length; i++)
```

Can be changed to:

```solidity
for (uint256 i; i < _references.length; i++)
```

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/IndexTemplate.sol#L493-L497

```solidity
if (totalLiquidity() > 0) {
    return (totalAllocatedCredit * MAGIC_SCALE_1E6) / totalLiquidity();
} else {
    return 0;
}
```

Can be changed to:

```solidity
if (totalLiquidity() > 0) {
    return (totalAllocatedCredit * MAGIC_SCALE_1E6) / totalLiquidity();
} 
```

