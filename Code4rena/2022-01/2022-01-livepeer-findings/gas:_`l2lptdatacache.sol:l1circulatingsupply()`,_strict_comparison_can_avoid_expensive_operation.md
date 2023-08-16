## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas: `L2LPTDataCache.sol:l1CirculatingSupply()`, strict comparison can avoid expensive operation](https://github.com/code-423n4/2022-01-livepeer-findings/issues/150) 

# Handle

Dravee


# Vulnerability details

## Impact
Increased gas cost (2 SLOADs and 1 SUB are avoided with the suggested solution)

## Proof of Concept
In `L2LPTDataCache.sol:l1CirculatingSupply()`, the code is as such:
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

Here, in the case of `l1TotalSupply == l2SupplyFromL1`, the substraction is equal to 0, but the computation is still done instead of return the already present 0 value. This could be avoided by making a strict comparison:
```
File: L2LPTDataCache.sol
91:         return
92:             l1TotalSupply > l2SupplyFromL1
93:                 ? l1TotalSupply - l2SupplyFromL1
94:                 : 0;
```

## Tools Used
VS Code

## Recommended Mitigation Steps
Use `>` instead of `>=`

