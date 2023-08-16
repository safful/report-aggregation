## Tags

- bug
- 0 (Non-critical)
- disagree with severity
- sponsor confirmed

# [Interest model is non-continuous](https://github.com/code-423n4/2021-07-wildcredit-findings/issues/112) 

# Handle

cmichel


# Vulnerability details

The `InterestRateModel.borrowRatePerBlock` function has a literal jump at target ratio and does not form a continuous function.
Usually (as in Compound) it's a piece-wise continuous function with a linear function `f` on `[0%; TARGET%]` and a second linear function `g` on [`TARGET%; 100%]` where `f(TARGET) = g(TARGET)` and `g`'s slope is much higher than `f` to discourage further borrows.

Example:
Assuming a `TARGET_UTILIZATION` of 80%, the `borrowRatePerBlock` for a utilisation ratio slightly less than `TARGET_UTILIZATION` (`if` branch) would be: `LOW_RATE * TARGET_UTILIZATION`.
However, when borrowing **at** `TARGET_UTILIZATION` (`else` branch), the `borrowRatePerBlock` suddenly becomes `TARGET_UTILIZATION`, i.e., a `(1-LOW_RATE) * TARGET_UTILIZATION` increase.

This is because `debt - (supply * TARGET_UTILIZATION / 100e18) = 0` (as `debt * 100e18 / supply = TARGET_UTILIZATION`) and thus the inner `utilization = 0`.


## Impact
Borrowing a single wei more that pushes the utilization ratio to the `TARGET_UTILIZATION` (going from `if` to `else` branch)  leads to suddenly having to pay 20% (1 - target) more interest **on the overall debt position**.

## Recommended Mitigation Steps
I think the expected behavior for the `else` case should be something like `TARGET_UTILIZATION * LOW_RATE + (HIGH_RATE - TARGET_UTILIZATION * LOW_RATE) * utilization / 100e18` such that it's a continuous function at utiisation ratio of `TARGET_UTILIZATION`.

