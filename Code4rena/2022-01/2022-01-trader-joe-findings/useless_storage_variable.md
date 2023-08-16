## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Useless storage variable](https://github.com/code-423n4/2022-01-trader-joe-findings/issues/116) 

# Handle

p4st13r4


# Vulnerability details

## Impact

`pendingRJoe()` reads a user into a storage variable, which is redundant since it’s a `view()` function and the variable is never modified in place. It can be replaced by a `memory` variable for readability

## Proof of Concept

[https://github.com/code-423n4/2022-01-trader-joe/blob/main/contracts/RocketJoeStaking.sol#L82](https://github.com/code-423n4/2022-01-trader-joe/blob/main/contracts/RocketJoeStaking.sol#L82)

## Tools Used

Editor

## Recommended Mitigation Steps

```jsx
UserInfo memory user = userInfo[_user];
```

