## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [AuctionBurnReserveSkew:getRealBurnBudget no underflow check needed](https://github.com/code-423n4/2021-11-malt-findings/issues/152) 

# Handle

GiveMeTestEther


# Vulnerability details

## Impact
After teh if statement on L74, we have premiumExcess <= maxBurnSpend and therefore don't need to do a save subtraction (underflow check) on L80.

## Proof of Concept
https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/AuctionBurnReserveSkew.sol#L69

## Tools Used

## Recommended Mitigation Steps
- rewrite L80 as: uint256 usableExcess = maxBurnSpend - premiumExcess;

