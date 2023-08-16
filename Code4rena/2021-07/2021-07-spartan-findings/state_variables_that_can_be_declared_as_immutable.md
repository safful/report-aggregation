## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [state variables that can be declared as immutable](https://github.com/code-423n4/2021-07-spartan-findings/issues/217) 

# Handle

JMukesh


# Vulnerability details

## Impact

https://docs.soliditylang.org/en/v0.8.6/contracts.html#constant-and-immutable-state-variables

## Proof of Concept

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Dao.sol#L15

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Pool.sol#L14

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Pool.sol#L18

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Synth.sol#L7
https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Synth.sol#L12

## Tools Used

manual review

## Recommended Mitigation Steps

