## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Avoid expensive storage reads in Parameters.sol](https://github.com/code-423n4/2022-01-insure-findings/issues/137) 

# Handle

p4st13r4


# Vulnerability details

## Impact

Many functions that read params, check whether the value is set for the given `target`, otherwise return the value for the zero-address. When doing this kind of check, the value of the `target` is read twice:

- once for checking if it’s set
- if it’s set, it’s read once more to read the actual params

These functions are used a lot of times inside all the contracts, so having them optimized as much as possible is required in order to save gas

## Proof of Concept

- [https://github.com/code-423n4/2022-01-insure/blob/main/contracts/Parameters.sol#L240](https://github.com/code-423n4/2022-01-insure/blob/main/contracts/Parameters.sol#L240)
- [https://github.com/code-423n4/2022-01-insure/blob/main/contracts/Parameters.sol#L271](https://github.com/code-423n4/2022-01-insure/blob/main/contracts/Parameters.sol#L271)
- [https://github.com/code-423n4/2022-01-insure/blob/main/contracts/Parameters.sol#L289](https://github.com/code-423n4/2022-01-insure/blob/main/contracts/Parameters.sol#L289)
- [https://github.com/code-423n4/2022-01-insure/blob/main/contracts/Parameters.sol#L313](https://github.com/code-423n4/2022-01-insure/blob/main/contracts/Parameters.sol#L313)
- [https://github.com/code-423n4/2022-01-insure/blob/main/contracts/Parameters.sol#L331](https://github.com/code-423n4/2022-01-insure/blob/main/contracts/Parameters.sol#L331)
- [https://github.com/code-423n4/2022-01-insure/blob/main/contracts/Parameters.sol#L343](https://github.com/code-423n4/2022-01-insure/blob/main/contracts/Parameters.sol#L343)
- [https://github.com/code-423n4/2022-01-insure/blob/main/contracts/Parameters.sol#L379](https://github.com/code-423n4/2022-01-insure/blob/main/contracts/Parameters.sol#L379)
- [https://github.com/code-423n4/2022-01-insure/blob/main/contracts/Parameters.sol#L397](https://github.com/code-423n4/2022-01-insure/blob/main/contracts/Parameters.sol#L397)

## Tools Used

Editor

## Recommended Mitigation Steps

Assign the target value and, if the check returns a value different from the zero-address, use it. For example, `getFeeRate` becomes:

```jsx
function getFeeRate(address _target)
    external
    view
    override
    returns (uint256)
{
    uint256 _targetFee = _fee[_target];
    if (_targetFee == 0) {
        return _fee[address(0)];
    } else {
        return _targetFee;
    }
}
```

