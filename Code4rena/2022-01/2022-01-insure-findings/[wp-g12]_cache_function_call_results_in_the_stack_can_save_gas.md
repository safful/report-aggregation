## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [[WP-G12] Cache function call results in the stack can save gas](https://github.com/code-423n4/2022-01-insure-findings/issues/231) 

# Handle

WatchPug


# Vulnerability details

Cache and reusing the function call results, instead of calling it again, can save gas from unnecessary code execution.

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/Vault.sol#L163-L173

```solidity
if (available() < _amount) { 
    //when USDC in this contract isn't enough
    uint256 _shortage = _amount - available();
    _unutilize(_shortage);

    assert(available() >= _amount);
}

balance -= _amount;
IERC20(token).safeTransfer(_to, _amount);
```

### Recommendation

Change to:

```solidity
uint256 availableAmount = available()
if ( availableAmount < _amount) { 
    //when USDC in this contract isn't enough
    uint256 _shortage = _amount - available();
    _unutilize(_shortage);

    assert(availableAmount >= _amount);
}

balance -= _amount;
IERC20(token).safeTransfer(_to, _amount);
```

Other examples include:

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/CDSTemplate.sol#L295-L304

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

`totalSupply()`

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/Vault.sol#L309-L312

```solidity
if (available() < _retVal) {
    uint256 _shortage = _retVal - available();
    _unutilize(_shortage);
}
```

`available()`

