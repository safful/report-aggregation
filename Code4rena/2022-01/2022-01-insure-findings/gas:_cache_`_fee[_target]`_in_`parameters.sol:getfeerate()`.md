## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas: Cache `_fee[_target]` in `Parameters.sol:getFeeRate()`](https://github.com/code-423n4/2022-01-insure-findings/issues/320) 

# Handle

Dravee


# Vulnerability details

## Impact
SLOADs are expensive

## Proof of Concept
Here, `_fee[_target]` can be loaded twice from storage:
```
271:     function getFeeRate(address _target)
272:         external
273:         view
274:         override
275:         returns (uint256)
276:     {
277:         if (_fee[_target] == 0) {
278:             return _fee[address(0)];
279:         } else {
280:             return _fee[_target];
281:         }
282:     }
```

## Tools Used
VS Code

## Recommended Mitigation Steps
Cache the storage reading in a memory variable

