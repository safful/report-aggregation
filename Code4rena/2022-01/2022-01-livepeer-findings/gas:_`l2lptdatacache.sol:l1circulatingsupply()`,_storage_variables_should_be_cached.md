## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas: `L2LPTDataCache.sol:l1CirculatingSupply()`, Storage variables should be cached](https://github.com/code-423n4/2022-01-livepeer-findings/issues/151) 

# Handle

Dravee


# Vulnerability details

## Impact
Increased gas cost

## Proof of Concept
In `L2LPTDataCache.sol:l1CirculatingSupply()`, the code is as follows:
```
File: L2LPTDataCache.sol
88:     function l1CirculatingSupply() public view returns (uint256) {
89:         // After the first update from L1, l1TotalSupply should always be >= l2SupplyFromL1
90:         // The below check is defensive to avoid reverting if this invariant for some reason violated
91:         return
92:             l1TotalSupply >= l2SupplyFromL1
93:                 ? l1TotalSupply - l2SupplyFromL1
94:                 : 0;
95:     }
```

I suspect that statistically, the arithmetic operation `l1TotalSupply - l2SupplyFromL1` should often be triggered. Therefore, caching the 2 variables `l1TotalSupply` and `l2SupplyFromL1` in memory variables would save the 2 SLOADs (~200 gas) in the substraction and cost 4 MLOADs (~12 gas) and 2 MSTOREs (6 gas).

It can be done this way, as an example: `(uint256 _l1TotalSupply, uint256 _l2SupplyFromL1) = (l1TotalSupply, l2SupplyFromL1);`

## Tools Used
VS Code

## Recommended Mitigation Steps
Cache `l1TotalSupply` and `l2SupplyFromL1` in local variables

