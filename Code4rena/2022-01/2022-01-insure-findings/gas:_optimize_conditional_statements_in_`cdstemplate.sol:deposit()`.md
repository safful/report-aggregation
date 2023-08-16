## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas: Optimize Conditional Statements in `CDSTemplate.sol:deposit()`](https://github.com/code-423n4/2022-01-insure-findings/issues/285) 

# Handle

Dravee


# Vulnerability details

## Impact
It's possible to save gas by optimizing the checks in conditional statements (`if`, `else if` and `else`). This would save a few opcodes and avoid redundant checks.

## Proof of Concept
In `CDSTemplate.sol:deposit()`, the code is as follows:
```
140:         if (_supply > 0 && _liquidity > 0) { 
141:             _mintAmount = (_amount * _supply) / _liquidity;
142:         } else if (_supply > 0 && _liquidity == 0) {
143:             //when vault lose all underwritten asset = 
144:             _mintAmount = _amount * _supply; //dilute LP token value af. Start CDS again.
145:         } else {
146:             //when _supply == 0,
147:             _mintAmount = _amount;
148:         }
```

The conditions checks can be optimized with the following (read the `@audit-info` comments for futher information):
```
        if (_supply == 0) { 
            _mintAmount = _amount;
        } else if (_liquidity == 0) { // @audit-info : implicit _supply > 0 as above condition is false
            //when vault lose all underwritten asset = 
            _mintAmount = _amount * _supply; //dilute LP token value af. Start CDS again.
        } else { // @audit-info : implicit _supply > 0 and _liquidity > 0 as both the previous conditions are false
            _mintAmount = (_amount * _supply) / _liquidity;
        }
```

## Tools Used
VS Code

## Recommended Mitigation Steps
Compact conditions in mentioned logic statements

