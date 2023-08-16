## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Debt accrual is path-dependant and inaccurate](https://github.com/code-423n4/2021-10-mochi-findings/issues/129) 

# Handle

cmichel


# Vulnerability details

The total `debt` in `MochiVault.accrueDebt` increases by the current `debt` times the debt index growth.
This is correct but the total `debt` is then _reduced_ again by the calling _user's_ discounted debt, meaning, the total debt depends on which specific user performs the debt accrual.

This should not be the case.

## POC
Assume we have a total debt of `2000`, two users A and B, where A has a debt of 1000, and B has a debt of 100.
The (previous) `debtIndex = 1.0` and accruing it now would increase it to `1.1`.

There's a difference if user A or B first does the accrual.

#### User A accrues first
User A calls `accrueDebt`: `increased = 2000 * 1.1/1.0 - 2000 = 200`. Thus `debts` is first set to `2200`. The user's `increasedDebt = 1000 * 1.1 / 1.0 - 1000 = 100` and assume a discount of `10%`, thus `discountedDebt = 100 * 10% = 10`.
Then `debts = 2200 - 10 = 2190`.

The next accrual will work with a total debt of `2190`.

#### User B accruess first
User B calls `accrueDebt`: `increased = 2000 * 1.1/1.0 - 2000 = 200`. Thus `debts` is first set to `2200`. The user's `increasedDebt = 100 * 1.1 / 1.0 - 100 = 10` and assume a discount of `10%`, thus `discountedDebt = 10 * 10% = 1`.
Then `debts = 2200 - 1 = 2199`.

The next accrual will work with a total debt of `2199`, leading to more debt overall.

## Impact
The total debt of a system depends on who performs the accruals which should ideally not be the case.
The discrepancy compounds and can grow quite large if a whale always does the accrual compared to someone with almost no debt or no discount.

## Recommended Mitigation Steps
Don't use the discounts or track the weighted average discount across all users that is subtracted from the increased total debt each time, i.e., reduce it by the discount of **all users** (instead of current caller only) when accruing to correctly track the debt.


