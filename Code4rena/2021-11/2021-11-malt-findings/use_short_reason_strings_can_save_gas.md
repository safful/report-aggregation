## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Use short reason strings can save gas](https://github.com/code-423n4/2021-11-malt-findings/issues/317) 

# Handle

WatchPug


# Vulnerability details

Every reason string takes at least 32 bytes.

Use short reason strings that fits in 32 bytes or it will become more expensive.

Instances include:


https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/Permissions.sol#L135-L141

```solidity=135
  function _notSameBlock() internal {
    require(
      block.number > lastBlock[_msgSender()],
      "Can't carry out actions in the same block"
    );
    lastBlock[_msgSender()] = block.number;
  }
```


https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/Auction.sol#L223-L223
```solidity=223
require(!auction.active, "Cannot claim tokens on an active auction");
```

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/Auction.sol#L230-L230
```solidity=230
require(redemption <= remaining.add(1), "Cannot claim more tokens than available");
```

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/Auction.sol#L659-L659
```solidity=659
require(auction.startingTime > 0, "No auction available for the given id");
```

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/AuctionEscapeHatch.sol#L177-L177
```solidity=177
require(!active, "Cannot exit early on an active auction");
```

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/AuctionParticipant.sol#L136-L136
```solidity=136
require(_index > replenishingIndex, "Cannot replenishingIndex to old value");
```

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/DAO.sol#L56-L56
```solidity=56
require(block.timestamp >= getEpochStartTime(epoch + 1), "Cannot advance epoch until start of new epoch");
```

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/ERC20Permit.sol#L81-L81
```solidity=81
require(balance >= value, "ERC20Permit: transfer amount exceeds balance");
```

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/ERC20Permit.sol#L122-L122
```solidity=122
require(balance >= value, "ERC20Permit: transfer amount exceeds balance");
```

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/Malt.sol#L65-L65
```solidity=65
require(_service != address(0), "Cannot use address 0 as transfer service");
```


https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/MovingAverage.sol#L412-L412
```solidity=412
require(_sampleLength > 0, "Cannot have 0 second sample length");
```

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/RewardSystem/RewardDistributor.sol#L154-L154
```solidity=154
require(forfeited <= _globals.declaredBalance, "Cannot forfeit more than declared");
```


https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/RewardSystem/RewardDistributor.sol#L275-L275
```solidity=275
require(amount <= _globals.declaredBalance, "Can't decrement more than total reward balance");
```


https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/Timelock.sol#L118-L121
```solidity=118
    require(
      eta >= block.timestamp.add(delay),
      "Timelock::queueTransaction: Estimated execution block must satisfy delay."
    );
```


https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/Timelock.sol#L165-L176
```solidity=165
    require(
      queuedTransactions[txHash],
      "Timelock::executeTransaction: Transaction hasn't been queued."
    );
    require(
      block.timestamp >= eta,
      "Timelock::executeTransaction: Transaction hasn't surpassed time lock."
    );
    require(
      block.timestamp <= eta.add(gracePeriod),
      "Timelock::executeTransaction: Transaction is stale."
    );
```

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/Timelock.sol#L195-L198
```solidity=195
    require(
      success,
      "Timelock::executeTransaction: Transaction execution reverted."
    );
```

