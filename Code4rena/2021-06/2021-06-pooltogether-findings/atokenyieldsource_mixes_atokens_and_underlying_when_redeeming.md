## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- ATokenYieldSource

# [ATokenYieldSource mixes aTokens and underlying when redeeming](https://github.com/code-423n4/2021-06-pooltogether-findings/issues/86) 

# Handle

cmichel


# Vulnerability details

The `ATokenYieldSource.redeemToken` function burns `aTokens` and sends out underlying, however, it's used in a reverse way in the code:
The `balanceDiff` is used as the `depositToken` that is transferred out but it's computed on the **aTokens** that were burned instead of on the `depositToken` received.

## Impact

It should not directly lead to issues as aTokens are 1-to-1 with their underlying but we still recommend doing it correctly to make the code more robust against any possible rounding issues.

## Recommended Mitigation Steps

Compute `balanceDiff` on the underyling balance (depositToken), not on the aToken.
Subtract the actual burned aTokens from the user shares.

