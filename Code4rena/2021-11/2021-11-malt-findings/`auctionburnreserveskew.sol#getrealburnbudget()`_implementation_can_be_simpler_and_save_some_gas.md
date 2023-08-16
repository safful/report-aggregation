## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [`AuctionBurnReserveSkew.sol#getRealBurnBudget()` Implementation can be simpler and save some gas](https://github.com/code-423n4/2021-11-malt-findings/issues/306) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/AuctionBurnReserveSkew.sol#L69-L89

```solidity=69{76,82-84}
  function getRealBurnBudget(
    uint256 maxBurnSpend,
    uint256 premiumExcess
  ) public view returns(uint256) {
    // Returning maxBurnSpend = maximum supply burn with no reserve ratio improvement
    // Returning premiumExcess = maximum reserve ratio improvement with no real supply burn

    if (premiumExcess > maxBurnSpend) {
      return premiumExcess;
    }

    uint256 usableExcess = maxBurnSpend.sub(premiumExcess);

    if (usableExcess == 0) {
      return premiumExcess;
    }

    uint256 burnable = consult(usableExcess);

    return premiumExcess + burnable;
  }
```

L82-84 `if (maxBurnSpend == premiumExcess)` can be combined with L76-78.

### Recommendation

Change to:

```solidity=69{76}
  function getRealBurnBudget(
    uint256 maxBurnSpend,
    uint256 premiumExcess
  ) public view returns(uint256) {
    // Returning maxBurnSpend = maximum supply burn with no reserve ratio improvement
    // Returning premiumExcess = maximum reserve ratio improvement with no real supply burn

    if (premiumExcess >= maxBurnSpend) {
      return premiumExcess;
    }

    uint256 usableExcess = maxBurnSpend.sub(premiumExcess);

    uint256 burnable = consult(usableExcess);

    return premiumExcess + burnable;
  }
```

