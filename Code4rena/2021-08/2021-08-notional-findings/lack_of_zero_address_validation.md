## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Lack of Zero Address Validation](https://github.com/code-423n4/2021-08-notional-findings/issues/93) 

# Handle

leastwood


# Vulnerability details

## Impact

There is currently no input validation done on the `Router.initialize()` and `NoteERC20.initialize()` functions, potentially leading to an initialized state where the contracts have no owner and the deployer needs to re-deploy the contract to have it working properly.

## Proof of Concept

https://github.com/code-423n4/2021-08-notional/blob/main/contracts/external/Router.sol#L63-L92
https://github.com/code-423n4/2021-08-notional/blob/main/contracts/external/governance/NoteERC20.sol#L90-L108

## Tools Used

Manual code review

## Recommended Mitigation Steps

Perform zero address checks for the `owner_`, `pauseRouter_` and `pauseGuardian_` inputs to ensure the contract isn't initialized into an unexpected state.

