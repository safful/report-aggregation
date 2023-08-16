## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Only use `SafeMath` when necessary can save gas](https://github.com/code-423n4/2021-11-malt-findings/issues/307) 

# Handle

WatchPug


# Vulnerability details

For the arithmetic operations that will never over/underflow, using SafeMath will cost more gas.

For example:

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/Auction.sol#L238-L242

```solidity=238
    if (amountTokens > unclaimedArbTokens) {
      unclaimedArbTokens = 0;
    } else {
      unclaimedArbTokens = unclaimedArbTokens.sub(amountTokens);
    }
```

`unclaimedArbTokens - amountTokens` will never underflow.

### Recommendation

Change to:

```solidity=238
    if (amountTokens >= unclaimedArbTokens) {
      unclaimedArbTokens = 0;
    } else {
      unclaimedArbTokens -= amountTokens;
    }
```



https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/AuctionBurnReserveSkew.sol#L76-L80

```solidity=76
if (premiumExcess > maxBurnSpend) {
  return premiumExcess;
}

uint256 usableExcess = maxBurnSpend.sub(premiumExcess);
```

`maxBurnSpend - premiumExcess` will never underflow.

### Recommendation

Change to:

```solidity=76
if (premiumExcess > maxBurnSpend) {
  return premiumExcess;
}

uint256 usableExcess = maxBurnSpend - premiumExcess;
```

