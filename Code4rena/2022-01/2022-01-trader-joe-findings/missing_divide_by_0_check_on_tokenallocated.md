## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Missing divide by 0 check on tokenAllocated](https://github.com/code-423n4/2022-01-trader-joe-findings/issues/140) 

# Handle

Dravee


# Vulnerability details

## Impact
A division by 0 could occur

## Proof of Concept
There are no checks that the denominator is `!= 0` here:
```
File: LaunchEvent.sol
392:         uint256 tokenAllocated = tokenReserve;
393: 
394:         // Adjust the amount of tokens sent to the pool if floor price not met
395:         if (
396:             floorPrice > (wavaxReserve * 10**token.decimals()) / tokenAllocated
397:         ) {
```

tokenReserve (`uint256 tokenAllocated = tokenReserve;`) can be equal to 0 according to this comment: https://github.com/code-423n4/2022-01-trader-joe/blob/main/contracts/LaunchEvent.sol#L106

Therefore this could happen

## Tools Used
VS Code

## Recommended Mitigation Steps
Check for `tokenAllocated != 0` before this division

