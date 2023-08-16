## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [False information given to the user](https://github.com/code-423n4/2021-11-fei-findings/issues/64) 

# Handle

Czar102


# Vulnerability details

## Impact
In `TribeRagequit` when a user tries to withdraw more than is left, a `"already ragequit all you tokens"` error is displayed. This is not necessarily true.\
For example, if one may withdraw multiplier 1000 times and calls `ngmi(...)` function, stating their balance as 1001 tokens, they would get an information that they have already ragequit all tokens, whic is false as non of them have been ragequit yet.

## Proof of Concept
[code](https://github.com/code-423n4/2021-11-fei/blob/add34324513b863f58e4ef7b3cd0c12d776dbb7f/contracts/TRIBERagequit.sol#L74)

## Tools Used
Manual analysis

## Recommended Mitigation Steps
Change the information to correctly reflect the situation.

