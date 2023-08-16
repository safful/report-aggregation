## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [[WP-G18] Avoiding repeated `marketStatus` checks can save gas](https://github.com/code-423n4/2022-01-insure-findings/issues/237) 

# Handle

WatchPug


# Vulnerability details

Check `marketStatus` before for loops can save gas from unnecessary repeated checks.

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/PoolTemplate.sol#L342-L365

```solidity
function unlockBatch(uint256[] calldata _ids) external {
    for (uint256 i = 0; i < _ids.length; i++) {
        unlock(_ids[i]);
    }
}

function unlock(uint256 _id) public {
    require(
        insurances[_id].status == true &&
            marketStatus == MarketStatus.Trading &&
            insurances[_id].endTime + parameters.getGrace(msg.sender) <
            block.timestamp,
        "ERROR: UNLOCK_BAD_COINDITIONS"
    );
    insurances[_id].status == false;

    lockedAmount = lockedAmount - insurances[_id].amount;

    emit Unlocked(_id, insurances[_id].amount);
}
```

### Recomandation

Change to:

```solidity
function unlockBatch(uint256[] calldata _ids) external {
    require(marketStatus == MarketStatus.Trading, "ERROR: UNLOCK_BAD_COINDITIONS")
    for (uint256 i = 0; i < _ids.length; i++) {
        _unlock(_ids[i]);
    }
}

function unlock(uint256 _id) external {
    require(marketStatus == MarketStatus.Trading, "ERROR: UNLOCK_BAD_COINDITIONS");
    _unlock(_id);
}

function _unlock(uint256 _id) internal {
    require(
        insurances[_id].status == true &&
            insurances[_id].endTime + parameters.getGrace(msg.sender) <
            block.timestamp,
        "ERROR: UNLOCK_BAD_COINDITIONS"
    );
    insurances[_id].status == false;

    lockedAmount = lockedAmount - insurances[_id].amount;

    emit Unlocked(_id, insurances[_id].amount);
}
```


