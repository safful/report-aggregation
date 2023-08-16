## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Underflow problems occurring when a token has >18 decimals](https://github.com/code-423n4/2021-06-tracer-findings/issues/116) 

# Handle

tensors


# Vulnerability details

## Impact
The contracts assume that all tokens will have <=18 decimals. If the Tracer team are the only people deploying the
contracts, and they keep this in mind, this isn't a problem. If the contracts are to be deployed by other people, this assumption should be made explicit and hard-coded.
 
## Proof of Concept
https://github.com/code-423n4/2021-06-tracer/blob/main/src/contracts/lib/LibBalances.sol#L220-L232
We can see that the scaler computations will underflow and be defined when it should not be.

## Recommended Mitigation Steps
Write a require check that ensures tokenDecimals <= 18 before running the above functions.

