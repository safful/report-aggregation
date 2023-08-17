## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [Checking yieldBearingToken against u instead of backingToken](https://github.com/code-423n4/2022-06-illuminate-findings/issues/139) 

# Lines of code

https://github.com/code-423n4/2022-06-illuminate/blob/912be2a90ded4a557f121fe565d12ec48d0c4684/lender/Lender.sol#L446


# Vulnerability details

## Impact
The lend function for tempus will fail with the right market.

## Proof of concept
checks `if (ITempus(principal).yieldBearingToken() != IERC20Metadata(u))`, while it should check `ITempus(principal).backingToken()`


## Recommendation
Do this instead:
```
    if (ITempus(principal).backingToken() != IERC20Metadata(u))
```

