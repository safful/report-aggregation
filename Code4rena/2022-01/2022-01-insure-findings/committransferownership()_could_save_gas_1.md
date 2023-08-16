## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [commitTransferOwnership() could save gas 1](https://github.com/code-423n4/2022-01-insure-findings/issues/139) 

# Handle

p4st13r4


# Vulnerability details

## Impact

When emitting the event, the function argument could be used, instead of reading from storage again

## Proof of Concept

[https://github.com/code-423n4/2022-01-insure/blob/main/contracts/Ownership.sol#L62](https://github.com/code-423n4/2022-01-insure/blob/main/contracts/Ownership.sol#L62)

## Tools Used

Editor

## Recommended Mitigation Steps

Change to:

```jsx
emit CommitNewOwnership(newOwner);
```

