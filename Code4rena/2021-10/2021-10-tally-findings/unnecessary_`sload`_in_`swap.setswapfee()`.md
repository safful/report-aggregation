## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Unnecessary `SLOAD` in `Swap.setSwapFee()`](https://github.com/code-423n4/2021-10-tally-findings/issues/63) 

# Handle

pants


# Vulnerability details

This line in `Swap.setSwapFee()` perfoms an `SLOAD` operation for a value that is already stored in a local variable:
```
emit NewSwapFee(swapFee);
```

## Impact
Storage reads are much more expensive than reading local variables.

## Tool Used
Manual code review.

## Recommended Mitigation Steps
Use the already existing local variable instead of loading this value from storage:
```
emit NewSwapFee(swapFee_);
```

