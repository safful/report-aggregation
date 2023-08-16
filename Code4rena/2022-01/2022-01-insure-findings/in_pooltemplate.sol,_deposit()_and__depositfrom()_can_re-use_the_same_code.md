## Tags

- bug
- 0 (Non-critical)
- resolved
- sponsor confirmed

# [In PoolTemplate.sol, deposit() and _depositFrom() can re-use the same code](https://github.com/code-423n4/2022-01-insure-findings/issues/133) 

# Handle

p4st13r4


# Vulnerability details

## Impact

The public `deposit` uses basically the same code of the internal `_depositFrom`. The only difference between them is that the former uses `msg.sender`, while the latter uses a parameter as `from` address. In order to minimize code duplication, `deposit` should be calling `_depositFrom` rather than being reimplemented using copy-paste

## Proof of Concept

[https://github.com/code-423n4/2022-01-insure/blob/main/contracts/PoolTemplate.sol#L232](https://github.com/code-423n4/2022-01-insure/blob/main/contracts/PoolTemplate.sol#L232)

[https://github.com/code-423n4/2022-01-insure/blob/main/contracts/PoolTemplate.sol#L255](https://github.com/code-423n4/2022-01-insure/blob/main/contracts/PoolTemplate.sol#L255)

## Tools Used

Editor

## Recommended Mitigation Steps

Write `deposit` like this:

```jsx
function deposit(uint256 _amount) public returns (uint256 _mintAmount) {
    _depositFrom(_amount, msg.sender);
}
```

