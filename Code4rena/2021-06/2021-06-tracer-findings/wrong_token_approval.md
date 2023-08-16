## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- disagree with severity

# [Wrong token approval](https://github.com/code-423n4/2021-06-tracer-findings/issues/99) 

# Handle

cmichel


# Vulnerability details

The pool holdings of `Insurance` (`publicCollateralAmount` and `bufferCollateralAmount`) is in WAD (18 decimals) but it's used as a raw token value in `drainPool`

```solidity
// amount is a mix of pool holdings, i.e., 18 decimals
// this requires amount to be in RAW! if tracerMarginToken has > 18 decimals, it'll break, < 18 decimals will approve too much
tracerMarginToken.approve(address(tracer), amount);
// this requires amount to be in WAD which is correct
tracer.deposit(amount);
```

## Impact

If `tracerMarginToken` has less than 18 decimals, the approval approves orders of magnitude more tokens than required for the `deposit` call that follows.
If `tracerMarginToken` has more than 18 decimals, the `deposit` that follows would fail as fewer tokens were approved, but the protocol seems to disallow tokens in general with more than 18 decimals.

## Recommended Mitigation Steps
Convert the `amount` to a "raw token value" and approve this one instead.


