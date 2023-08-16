## Tags

- bug
- disagree with severity
- 3 (High Risk)
- sponsor confirmed
- filed

# [Wrong liquidity units calculation](https://github.com/code-423n4/2021-04-vader-findings/issues/204) 

# Handle

@cmichelio


# Vulnerability details


## Vulnerability Details

The spec defines the number of LP units to be minted as `units = (P (a B + A b))/(2 A B) * slipAdjustment = P * (part1 + part2) / part3 * slipAdjustments` but the `Utils.calcLiquidityUnits` function computes `((P * part1) + part2) / part3 * slipAdjustments`.
The associativity on `P * part1` is wrong, and `part2` is not multiplied by `P`.

## Impact

The math from the spec is not correclty implemented and could lead to the protocol being economically exploited, as redeeming the minted LP tokens does not result in the initial tokens anymore.

## Recommended Mitigation Steps

Fix the equation.


