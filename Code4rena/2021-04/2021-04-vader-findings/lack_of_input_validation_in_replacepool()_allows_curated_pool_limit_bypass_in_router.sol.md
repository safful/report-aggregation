## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Lack of input validation in replacePool() allows curated pool limit bypass in Router.sol](https://github.com/code-423n4/2021-04-vader-findings/issues/87) 

# Handle

0xRajeev


# Vulnerability details

## Impact

There is no input validation in replacePool() function to check if oldToken exists and is curated. Using a non-existing oldToken (even 0 address) passes the check on L236 (because Pools.getBaseAmount() will return 0 for the non-existing token) and newToken will be made curated. This can be used to bypass the curatedPoolLimit enforced only in curatePool() function.

## Proof of Concept

https://github.com/code-423n4/2021-04-vader/blob/3041f20c920821b89d01f652867d5207d18c8703/vader-protocol/contracts/Router.sol#L234-L241

https://github.com/code-423n4/2021-04-vader/blob/3041f20c920821b89d01f652867d5207d18c8703/vader-protocol/contracts/Pools.sol#L227-L229

https://github.com/code-423n4/2021-04-vader/blob/3041f20c920821b89d01f652867d5207d18c8703/vader-protocol/contracts/Router.sol#L227

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Check if oldToken exists and is curated as part of input validation in replacePool() function.


