## Tags

- bug
- disagree with severity
- QA (Quality Assurance)
- resolved
- sponsor confirmed

# [Used deprecated methods](https://github.com/code-423n4/2022-06-yieldy-findings/issues/42) 

# Lines of code

https://github.com/code-423n4/2022-06-yieldy/blob/8400e637d9259b7917bde259a5a2fbbeb5946d45/src/contracts/Yieldy.sol#L38
https://github.com/code-423n4/2022-06-yieldy/blob/8400e637d9259b7917bde259a5a2fbbeb5946d45/src/contracts/Yieldy.sol#L61-L62


# Vulnerability details

## Impact
A deprecated method is used in `Yieldy` contract.

## Proof of Concept

In `Yieldy` contract the method `_setupRole` is used, and and it is explicitly marked as deprecated by OpenZeppelin.

> * NOTE: This function is deprecated in favor of {_grantRole}.

Affected source code:

- [Yieldy.sol#L38](https://github.com/code-423n4/2022-06-yieldy/blob/8400e637d9259b7917bde259a5a2fbbeb5946d45/src/contracts/Yieldy.sol#L38)
- [Yieldy.sol#L61-L62](https://github.com/code-423n4/2022-06-yieldy/blob/8400e637d9259b7917bde259a5a2fbbeb5946d45/src/contracts/Yieldy.sol#L61-L62)

## Recommended Mitigation Steps
- The method `_setupRole` must be changed to `_grantRole`.

