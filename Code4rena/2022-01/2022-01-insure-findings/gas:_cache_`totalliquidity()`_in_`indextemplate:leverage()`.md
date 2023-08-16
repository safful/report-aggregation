## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas: Cache `totalLiquidity()` in `IndexTemplate:leverage()`](https://github.com/code-423n4/2022-01-insure-findings/issues/301) 

# Handle

Dravee


# Vulnerability details

## Impact
SLOADs are expensive

## Proof of Concept
Here, `totalLiquidity()` is loaded twice from storage
```
491:     function leverage() public view returns (uint256 _rate) {
492:         //check current leverage rate
493:         if (totalLiquidity() > 0) {
494:             return (totalAllocatedCredit * MAGIC_SCALE_1E6) / totalLiquidity();
495:         } else {
496:             return 0;
497:         }
498:     }
```

## Tools Used
VS Code

## Recommended Mitigation Steps
Cache `totalLiquidity()` in a variable

