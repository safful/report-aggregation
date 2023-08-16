## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas: Optimize Conditional Statements in `PoolTemplate.sol:worth()`](https://github.com/code-423n4/2022-01-insure-findings/issues/355) 

# Handle

Dravee


# Vulnerability details

## Impact
It's possible to save gas by optimizing the checks in conditional statements (`if`, `else if` and `else`). This would save a few opcodes and avoid redundant checks.

## Proof of Concept
In `PoolTemplate.sol:worth()`, the code is as follows:
```
799:     function worth(uint256 _value) public view returns (uint256 _amount) {
800:         uint256 _supply = totalSupply();
801:         uint256 _originalLiquidity = originalLiquidity();
802:         if (_supply > 0 && _originalLiquidity > 0) {
803:             _amount = (_value * _supply) / _originalLiquidity;
804:         } else if (_supply > 0 && _originalLiquidity == 0) {
805:             _amount = _value * _supply;
806:         } else {
807:             _amount = _value;
808:         }
809:     }
```

The conditions checks can be optimized with the following (read the `@audit-info` comments for further information):
```
    function worth(uint256 _value) public view returns (uint256 _amount) {
        uint256 _supply = totalSupply();
        uint256 _originalLiquidity = originalLiquidity();
        if (_supply == 0) {
            _amount = _value;
        } else if (_originalLiquidity == 0) {
            _amount = _value * _supply;
        } else {
            _amount = (_value * _supply) / _originalLiquidity;
        }
    }
```

## Tools Used
VS Code

## Recommended Mitigation Steps
Compact conditions in mentioned logic statements


