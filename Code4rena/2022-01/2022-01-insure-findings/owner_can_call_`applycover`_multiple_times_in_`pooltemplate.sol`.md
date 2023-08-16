## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Owner can call `applyCover` multiple times in `PoolTemplate.sol`](https://github.com/code-423n4/2022-01-insure-findings/issues/160) 

# Handle

camden


# Vulnerability details

## Impact
The owner could potentially extend the insurance period indefinitely in the `applyCover` function without ever allowing the market to resume. This is because there is no check in `applyCover` to ensure that the market is in a `Trading` state.

This can also allow the owner to emit fraudulent `MarketStatusChanged` events.

## Recommended Mitigation Steps
Require that the market be in a `Trading` state to allow another `applyCover` call.

