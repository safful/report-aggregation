## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- resolved
- fixed-in-upstream-repo

# [confusing comments](https://github.com/code-423n4/2021-08-floatcapital-findings/issues/12) 

# Handle

gpersoon


# Vulnerability details

## Impact
I've seen comments which are confusing:
~10^31 or 10 Trillion (10^13) ==> probably should be 2^31
x * 5e17` == `(x * 10e18) / 2`   ==> probably should be 1e18/2

## Proof of Concept
//https://github.com/code-423n4/2021-08-floatcapital/blob/main/contracts/contracts/Staker.sol#L19
// 2^52 ~= 4.5e15
  // With an exponent of 5, the largest total liquidity possible in a market (to avoid integer overflow on exponentiation) is ~10^31 or 10 Trillion (10^13)

//https://github.com/code-423n4/2021-08-floatcapital/blob/main/contracts/contracts/Staker.sol#L480
      // NOTE: `x * 5e17` == `(x * 10e18) / 2`

## Tools Used

## Recommended Mitigation Steps
Double check the comments

