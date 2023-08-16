## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Cheaper operation should be done first in an if statement](https://github.com/code-423n4/2022-01-trader-joe-findings/issues/319) 

# Handle

pedroais


# Vulnerability details

## Impact
Save gas
## Proof of Concept
The cheaper operation should be done first to save gas . auctionStart == 0 is cheaper than block.timestamp < auctionStart 
https://github.com/code-423n4/2022-01-trader-joe/blob/a1579f6453bc4bf9fb0db9c627beaa41135438ed/contracts/LaunchEvent.sol#L291




