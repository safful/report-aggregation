## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [Incorrect Event Emitted in Alchemist.sol](https://github.com/code-423n4/2021-11-yaxis-findings/issues/7) 

# Handle

TimmyToes


# Vulnerability details

## Impact
The event emitted is for the updating of a different fee (the harvest fee). This could
cause potential issues for any system wishing to integrate with yAxis and wishing to monitor
changes to the system and potentially react to them. Such a system could record the wrong harvest 
fee and would be unaware of updates to the borrow fee.

## Proof of Concept
https://github.com/code-423n4/2021-11-yaxis/blob/0311dd421fb78f4f174aca034e8239d1e80075fe/contracts/v3/alchemix/Alchemist.sol#L299
Is the same as line 284

## Recommended Mitigation Steps
Create a new event:
event BorrowFeeUpdated(uint256 borrowfee);
and call it on line 299 instead of HarvestFeeUpdated


