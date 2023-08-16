## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Simple interest calculation is not exact](https://github.com/code-423n4/2022-02-anchor-findings/issues/41) 

# Lines of code

https://github.com/code-423n4/2022-02-anchor/blob/7af353e3234837979a19ddc8093dc9ad3c63ab6b/contracts%2Fmoney-market-contracts%2Fcontracts%2Fmarket%2Fsrc%2Fborrow.rs#L304


# Vulnerability details

## Impact
The borrow rate uses a simple interest formula to compute the accrued debt, instead of a compounding formula.

```rust
pub fn compute_interest_raw(
    state: &mut State,
    block_height: u64,
    balance: Uint256,
    aterra_supply: Uint256,
    borrow_rate: Decimal256,
    target_deposit_rate: Decimal256,
) {
  // @audit simple interest
    let passed_blocks = Decimal256::from_uint256(block_height - state.last_interest_updated);

    let interest_factor = passed_blocks * borrow_rate;
    let interest_accrued = state.total_liabilities * interest_factor;
    // ...
}
```

This means the actual borrow rate and interest for suppliers depend on how often updates are made.
This difference should be negligible in highly active markets, but it could lead to a lower borrow rate in low-activity markets, leading to suppliers losing out on interest.

## Recommended Mitigation Steps
Ensure that the markets are accrued regularly, or switch to a compound interest formula (which has a higher computational cost due to exponentiation, but can be approximated, see Aave).


