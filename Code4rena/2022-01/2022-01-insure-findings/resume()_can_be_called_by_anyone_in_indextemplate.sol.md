## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [resume() can be called by anyone in IndexTemplate.sol](https://github.com/code-423n4/2022-01-insure-findings/issues/129) 

# Handle

p4st13r4


# Vulnerability details

## Impact

The `resume` function can be called by any user, at any time, even when the Index contract is not locked. There should be a check preventing it from being called unless the contract is `locked`

## Proof of Concept

[https://github.com/code-423n4/2022-01-insure/blob/main/contracts/IndexTemplate.sol#L459](https://github.com/code-423n4/2022-01-insure/blob/main/contracts/IndexTemplate.sol#L459)

## Tools Used

Editor

## Recommended Mitigation Steps

Add a require on top:

```jsx
require(locked);
```

