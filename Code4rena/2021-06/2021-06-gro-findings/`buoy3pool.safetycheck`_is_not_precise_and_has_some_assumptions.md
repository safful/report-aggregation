## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- disagree with severity

# [`Buoy3Pool.safetyCheck` is not precise and has some assumptions](https://github.com/code-423n4/2021-06-gro-findings/issues/104) 

# Handle

cmichel


# Vulnerability details

## Vulnerability Details
The `safetyCheck` function has several issues that impact how precise the checks are:

1. only checks if the `a/b` and `a/c` ratios are within `BASIS_POINTS`.
By transitivity `b/c` is only within `2 * BASIS_POINTS` if `a/b` and `a/c` are in range.
For a more precise check whether both USDC and USDT are within range, `b/c` must be checked as well.

2. If `a/b` is within range, this does not imply that `b/a` is within range.
> "inverted ratios, a/b bs b/a, while producing different results should both reflect the same change in any one of the two underlying assets, but in opposite directions"

Example: `lastRatio = 1.0`
`ratio: a = 1.0, b = 0.8` => `a/b = 1.25`, `b/a = 0.8`
If `a/b` was used with a 20% range, it'd be out of range, but `b/a` is in range.

3. The natspec for the function states that it checks Curve and an external oracle, but no external oracle calls are checked, both `_ratio` and `lastRatio` are only from Curve. Only `_updateRatios` checks the oracle.

## Recommended Mitigation Steps
In addition, check if `b/c` is within `BASIS_POINTS`.

