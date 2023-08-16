## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Cannot use most piecewise linear functions with current implementation](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/200) 

# Handle

cmichel


# Vulnerability details

The `ThreePieceWiseLinearPriceCurve.adjustParams` function uses three functions `f1, f2, f3` where `y_i = f_i(x_i)`.
It computes the y-axis intersect (`b2 = f_2(0), b3 = f_3(0)`) for each of these but uses **unsigned integers** for this, which means these values cannot become negative.
This rules out a whole class of functions, usually the ones that are desirable.

#### Example:
Check out this two-piece linear interest curve of Aave:

![Aave](https://docs.aave.com/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-M51Fy3ipxJS-0euJX3h-2670852272%2Fuploads%2Fycd9OMRnInNeetUa7Lj1%2FScreenshot%202021-11-23%20at%2018.52.26.png?alt=media&token=7a25b900-7023-4ee5-b582-367d56d31894)
The intersection of the second steep straight line with the y-axis `b_2 = f_2(0)` would be negative.

Example:
Imagine a curve that is flat at `10%` on the first 50% utilization but shoots up to `110%` at 100% utilization.
 - `m1 = 0, b1 = 10%, cutoff1 = 50%`
 - `m2 = 200%` => `b2 = m1 * cutoff1 + b1 - m2 * cutoff1 = f1(cutoff1) - m2 * cutoff1 = 10% - 200% * 50% = 10% - 100% = -90%`. (`f2(100%) = 200% * 100% - 90% = 110%` ✅)
This function would revert in the `b2` computation as it underflows due to being a negative value.

## Impact
Most curves that are actually desired for a lending platform (becoming steeper at higher utilization) cannot be used.

## Recommended Mitigation Steps
Evaluate the piecewise linear function in a different way that does not require computing the y-axis intersection value.
For example, for `cutoff2 >= x > cutoff1`, use `f(x) = f_1(cutoff) + f_2(x - cutoff)`.
See [Compound](https://github.com/compound-finance/compound-protocol/blob/master/contracts/JumpRateModel.sol#L85).


