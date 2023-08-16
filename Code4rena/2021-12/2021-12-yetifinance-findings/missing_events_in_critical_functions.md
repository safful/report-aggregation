## Tags

- bug
- 0 (Non-critical)
- disagree with severity
- sponsor confirmed

# [Missing events in critical functions](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/84) 

# Handle

SolidityScan


# Vulnerability details

## Impact
Events are important and should be emitted for tracking this off-chain for all important functions. 

## Proof of Concept
1. The function "updateTeamAddress" is used to update the team's address but we can notice no event is emitted. 
https://github.com/code-423n4/2021-12-yetifinance/blob/main/packages/contracts/contracts/TeamAllocation.sol#L81-L83

2. The same in function "updateTeamWallet" at 
https://github.com/code-423n4/2021-12-yetifinance/blob/main/packages/contracts/contracts/YetiFinanceTreasury.sol#L28-L30

## Tools Used

## Recommended Mitigation Steps
Add an event to these important functions where address updation is happening. This can also be marked as indexed event for better off-chain tracking

