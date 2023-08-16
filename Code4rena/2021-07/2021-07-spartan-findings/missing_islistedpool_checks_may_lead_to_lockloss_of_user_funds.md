## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Missing isListedPool checks may lead to lock/loss of user funds](https://github.com/code-423n4/2021-07-spartan-findings/issues/130) 

# Handle

0xRajeev


# Vulnerability details

## Impact

This isListedPool check implemented by isPool() is missing in many functions of the contract that accept pool/token addresses from users. getPool() returns the default mapping value of 0 for token that do not have valid pools. This lack of input validation may lead to use of zero/invalid pool addresses in the protocol context and reverts in the best case or burn/loss of user funds in the worst case.

## Proof of Concept

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/poolFactory.sol#L119-L133

Use of getPool() without isPool() check:
https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/BondVault.sol#L108

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Router.sol#L52

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Router.sol#L81

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Router.sol#L139

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Router.sol#L155

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Router.sol#L175

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Router.sol#L232

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Router.sol#L247-L248

Several usages of getPool() in Utils.sol and other places.

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Combine isPool() isListedPool check to getPool() so that it always returns a valid/listed pool in the protocol.

