## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [Inconsistent divide by 0 checks for `totalSupply()`](https://github.com/code-423n4/2022-01-insure-findings/issues/287) 

# Handle

Dravee


# Vulnerability details

## Impact
A division by 0 could occur

## Proof of Concept
While at some places, a check is made to make sure that `totalSupply() > 0`, it's not consistently the case, such as in the following places: 
```
contracts\CDSTemplate.sol:235:        _retVal = (vault.attributionValue(crowdPool) * _amount) / totalSupply();
contracts\CDSTemplate.sol:318:                _balance * vault.attributionValue(crowdPool) / totalSupply();
contracts\IndexTemplate.sol:216:        _retVal = (_liquidty * _amount) / totalSupply();
contracts\IndexTemplate.sol:530:            return (_balance * totalLiquidity()) / totalSupply();
contracts\PoolTemplate.sol:768:            return (_balance * originalLiquidity()) / totalSupply();
```

At the following places, the check is indeed made:
```
contracts\IndexTemplate.sol:514:            return (totalLiquidity() * MAGIC_SCALE_1E6) / totalSupply();
contracts\PoolTemplate.sol:747:            return (originalLiquidity() * MAGIC_SCALE_1E6) / totalSupply();
```

## Tools Used
VS Code

## Recommended Mitigation Steps
If this check is at least made at some places, this means that `totalSupply()` can indeed take a value of 0. Therefore, a check should always be made to prevent the div by 0

