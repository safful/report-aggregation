## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Dont calculate progressionBps, when not needed](https://github.com/code-423n4/2021-11-malt-findings/issues/288) 

# Handle

0x0x0x


# Vulnerability details

## Concept

[https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/AuctionEscapeHatch.sol#L200-L212](https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/AuctionEscapeHatch.sol#L200-L212):

```
		uint256 progressionBps = (block.timestamp - auctionEndTime) * 10000 / cooloffPeriod;
    if (progressionBps > 10000) {
      progressionBps = 10000;
    }

    if (fullReturn > amount) {
      // Allow a % of profit to be realised
      uint256 maxProfit = (fullReturn - amount) * (maxEarlyExitBps * progressionBps / 10000) / 1000;
      uint256 desiredReturn = amount + maxProfit;
      maltQuantity = desiredReturn.mul(pegPrice) / currentPrice;
    } 

    return maltQuantity;
```

`progressionBps` is only used, if there is a profit. Calculations of this parameter should be under if statement checking whether there is a profit to save gas and increase readability as follows:

```

    if (fullReturn > amount) {
      // Allow a % of profit to be realised
			uint256 progressionBps = (block.timestamp - auctionEndTime) * 10000 / cooloffPeriod;
	    if (progressionBps > 10000) {
	      progressionBps = 10000;
	    }
      uint256 maxProfit = (fullReturn - amount) * (maxEarlyExitBps * progressionBps / 10000) / 1000;
      uint256 desiredReturn = amount + maxProfit;
      maltQuantity = desiredReturn.mul(pegPrice) / currentPrice;
    } 

    return maltQuantity;
```

