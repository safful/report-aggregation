## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Bypass zap timelock](https://github.com/code-423n4/2021-12-nftx-findings/issues/178) 

# Handle

gzeon


# Vulnerability details

## Impact
The default value of `inventoryLockTime` in `NFTXStakingZap` is `7 days` while `DEFAULT_LOCKTIME` in `NFTXInventoryStaking` is 2 ms. These timelock value are used in `NFTXInventoryStaking` to eventually call `_timelockMint` in `XTokenUpgradeable`.

https://github.com/code-423n4/2021-12-nftx/blob/194073f750b7e2c9a886ece34b6382b4f1355f36/nftx-protocol-v2/contracts/solidity/token/XTokenUpgradeable.sol#L74
```
    function _timelockMint(address account, uint256 amount, uint256 timelockLength) internal virtual {
        uint256 timelockFinish = block.timestamp + timelockLength;
        timelock[account] = timelockFinish;
        emit Timelocked(account, timelockFinish);
        _mint(account, amount);
    }
```

The applicable timelock is calculated by `block.timestamp + timelockLength`, even when the existing timelock is further in the future. Therefore, one can reduce their long (e.g. 7 days) timelock to 2 ms calling `deposit` in `NFTXInventoryStaking`

## Proof of Concept
https://github.com/code-423n4/2021-12-nftx/blob/194073f750b7e2c9a886ece34b6382b4f1355f36/nftx-protocol-v2/contracts/solidity/NFTXStakingZap.sol#L160
https://github.com/code-423n4/2021-12-nftx/blob/194073f750b7e2c9a886ece34b6382b4f1355f36/nftx-protocol-v2/contracts/solidity/NFTXInventoryStaking.sol#L30

## Recommended Mitigation Steps
```
    function _timelockMint(address account, uint256 amount, uint256 timelockLength) internal virtual {
        uint256 timelockFinish = block.timestamp + timelockLength;
        if(timelockFinish > timelock[account]){
            timelock[account] = timelockFinish;
            emit Timelocked(account, timelockFinish);
        }
        _mint(account, amount);
    }
```

