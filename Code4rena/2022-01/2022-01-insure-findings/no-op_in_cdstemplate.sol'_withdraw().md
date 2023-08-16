## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [No-op in CDSTemplate.sol' withdraw()](https://github.com/code-423n4/2022-01-insure-findings/issues/130) 

# Handle

p4st13r4


# Vulnerability details

## Impact

The amount of the withdrawal request is not correctly updated after a withdrawal in `CDSTemplate.sol`. This happens because the withdrawal request is read from storage and put in memory, like this:

```jsx
Withdrawal memory request = withdrawalReq[msg.sender];
```

However, the requested amount is not updated properly since the `withdrawalReq` in the storage is never updated. Instead, its in-memory version is updated, but it’s useless because that object is never used again:

```jsx
//reduce requested amount
request.amount -= _amount;
```

This issue is non critical because there is a function that takes care of updating the withdrawal requests’ amount on every token transfer: [https://github.com/code-423n4/2022-01-insure/blob/main/contracts/CDSTemplate.sol#L358](https://github.com/code-423n4/2022-01-insure/blob/main/contracts/CDSTemplate.sol#L358)

The issue lies in the fact that the code seems to behave differently from how it looks at a first glance. Furthermore, the other two templates correctly update the value of the withdrawal request, so the version in `CDSTemplate.sol` should be aligned as well:

- [https://github.com/code-423n4/2022-01-insure/blob/main/contracts/IndexTemplate.sol#L239](https://github.com/code-423n4/2022-01-insure/blob/main/contracts/IndexTemplate.sol#L239)
- [https://github.com/code-423n4/2022-01-insure/blob/main/contracts/PoolTemplate.sol#L327](https://github.com/code-423n4/2022-01-insure/blob/main/contracts/PoolTemplate.sol#L327)

## Proof of Concept

[https://github.com/code-423n4/2022-01-insure/blob/main/contracts/CDSTemplate.sol#L230](https://github.com/code-423n4/2022-01-insure/blob/main/contracts/CDSTemplate.sol#L230)

## Tools Used

Editor

## Recommended Mitigation Steps

Update the `amount` of the current withdrawal request as well

