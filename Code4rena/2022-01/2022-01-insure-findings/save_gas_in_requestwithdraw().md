## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Save gas in requestWithdraw()](https://github.com/code-423n4/2022-01-insure-findings/issues/142) 

# Handle

p4st13r4


# Vulnerability details

## Impact

Users that incorrectly ask for a withdrawal equal to zero, will waste more gas (a storage read) since the check for `amount > 0` is put after the check for the available amount

## Proof of Concept

- [https://github.com/code-423n4/2022-01-insure/blob/main/contracts/CDSTemplate.sol#L191](https://github.com/code-423n4/2022-01-insure/blob/main/contracts/CDSTemplate.sol#L191)
- [https://github.com/code-423n4/2022-01-insure/blob/main/contracts/IndexTemplate.sol#L199](https://github.com/code-423n4/2022-01-insure/blob/main/contracts/IndexTemplate.sol#L199)
- [https://github.com/code-423n4/2022-01-insure/blob/main/contracts/PoolTemplate.sol#L282](https://github.com/code-423n4/2022-01-insure/blob/main/contracts/PoolTemplate.sol#L282)

## Tools Used

Editor

## Recommended Mitigation Steps

Move this require at the top of the `requestWithdraw` function:

```jsx
require(_amount > 0, "ERROR: REQUEST_ZERO");
```

