## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas Optimisation - Simplify `_atPhase()` Logic](https://github.com/code-423n4/2022-01-trader-joe-findings/issues/162) 

# Handle

kirk-baird


# Vulnerability details

## Impact

The logic in [_atPhase()`](https://github.com/code-423n4/2022-01-trader-joe/blob/main/contracts/LaunchEvent.sol#L590) can be simplified to save gas and code complexity.

The code can be simplified to the follwoing.

```solidity
    function _atPhase(Phase _phase) internal view {
            require(currentPhase() == _phase, "LaunchEvent: incorrect phase");
    }
```

## Proof of Concept

n/a

## Tools Used

n/a

## Recommended Mitigation Steps

Consider updating the code to that procided above.

