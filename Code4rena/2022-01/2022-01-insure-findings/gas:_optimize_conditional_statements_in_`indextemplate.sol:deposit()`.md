## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas: Optimize Conditional Statements in `IndexTemplate.sol:deposit()`](https://github.com/code-423n4/2022-01-insure-findings/issues/300) 

# Handle

Dravee


# Vulnerability details

## Impact
It's possible to save gas by optimizing the checks in conditional statements (`if`, `else if` and `else`). This would save a few opcodes and avoid redundant checks.

## Proof of Concept
In `IndexTemplate.sol:deposit()`, the code is as follows:
```
172:         if (_supply > 0 && _totalLiquidity > 0) {  
173:             _mintAmount = (_amount * _supply) / _totalLiquidity;
174:         } else if (_supply > 0 && _totalLiquidity == 0) {
175:             //when
176:             _mintAmount = _amount * _supply;
177:         } else {
178:             _mintAmount = _amount;
179:         }
```

The conditions checks can be optimized with the following (read the `@audit-info` comments for further information):
```
      if (_supply == 0) {
          _mintAmount = _amount;
      } else if (_totalLiquidity == 0) { // @audit-info : implicit _supply > 0 as above condition is false
          _mintAmount = _amount * _supply;
      } else { // @audit-info : implicit _supply > 0 and _totalLiquidity > 0 as both the previous conditions are false
          _mintAmount = (_amount * _supply) / _totalLiquidity;
      }
```

## Tools Used
VS Code

## Recommended Mitigation Steps
Compact conditions in mentioned logic statements


