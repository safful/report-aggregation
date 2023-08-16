## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Incorrect event parameter logs zero address instead of WBNB](https://github.com/code-423n4/2021-07-spartan-findings/issues/135) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The token argument used in CreatePool event emit of createPoolADD() should really be _token so that WBNB address is logged in the event instead of zero address when token == 0. Logging a zero address could confuse off-chain user interfaces because it is treated as a burn address by convention.

## Proof of Concept

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/poolFactory.sol#L60

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/poolFactory.sol#L49


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Use _token instead of token in event emit.

