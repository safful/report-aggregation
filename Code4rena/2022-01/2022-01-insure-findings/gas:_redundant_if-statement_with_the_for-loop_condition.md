## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas: Redundant if-statement with the for-loop condition](https://github.com/code-423n4/2022-01-insure-findings/issues/81) 

# Handle

Dravee


# Vulnerability details

## Impact
Increased gas cost

## Proof of Concept
In `Factory.sol`, the following `> 0` checks are redundant with the for-loop condition, because if `_references.length == 0` or `_conditions.length == 0`, the condition `uint256 i = 0; i <(_conditions)|(_references).length` will never be satisfied and the for-loop won't iterate:
```
175:         if (_references.length > 0) {
176:             for (uint256 i = 0; i < _references.length; i++) {
177:                 require(
178:                     reflist[address(_template)][i][_references[i]] == true ||
179:                         reflist[address(_template)][i][address(0)] == true,
180:                     "ERROR: UNAUTHORIZED_REFERENCE"
181:                 );
182:             }
183:         }
184: 
185:         if (_conditions.length > 0) {
186:             for (uint256 i = 0; i < _conditions.length; i++) {
187:                 if (conditionlist[address(_template)][i] > 0) {
188:                     _conditions[i] = conditionlist[address(_template)][i];
189:                 }
190:             }
191:         }

```

## Tools Used
VS Code

## Recommended Mitigation Steps
Remove these 2 if-statements

