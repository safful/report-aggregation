## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas: Use `else if` to save gas and simplify code](https://github.com/code-423n4/2022-01-insure-findings/issues/83) 

# Handle

Dravee


# Vulnerability details

## Impact
Increased gas cost

## Proof of Concept
In `IndexTemplate.sol:_adjustAlloc()`, the 3 following conditions are always evaluated:
```
                //Withdraw or Deposit credit
                if (_current > _target && _available != 0) {
                    //if allocated credit is higher than the target, try to decrease
                    uint256 _decrease = _current - _target;
                    IPoolTemplate(_poolList[i].addr).withdrawCredit(_decrease);
                    totalAllocatedCredit -= _decrease;
                }
                if (_current < _target) {
                    uint256 _allocate = _target - _current;
                    IPoolTemplate(_poolList[i].addr).allocateCredit(_allocate);
                    totalAllocatedCredit += _allocate;
                }
                if (_current == _target) {
                    IPoolTemplate(_poolList[i].addr).allocateCredit(0);
                }
```
The code can be optimized to save some gas:
```
                if (_current == _target) {
                    IPoolTemplate(_poolList[i].addr).allocateCredit(0);
                } else if (_current < _target) {
                    uint256 _allocate = _target - _current;
                    IPoolTemplate(_poolList[i].addr).allocateCredit(_allocate);
                    totalAllocatedCredit += _allocate;
                } else if (_current > _target && _available != 0) {
                    //Withdraw or Deposit credit
                    //if allocated credit is higher than the target, try to decrease
                    uint256 _decrease = _current - _target;
                    IPoolTemplate(_poolList[i].addr).withdrawCredit(_decrease);
                    totalAllocatedCredit -= _decrease;
                }
```

## Tools Used
VS Code

## Recommended Mitigation Steps
Apply the refacto

