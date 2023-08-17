## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed
- reviewed

# [`_decimalMultiplier` doesn't account for tokens with decimals higher than 18](https://github.com/code-423n4/2022-04-backd-findings/issues/49) 

# Lines of code

https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/contracts/strategies/StrategySwapper.sol#L287-L289
https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/contracts/strategies/StrategySwapper.sol#L318-L320
https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/contracts/strategies/StrategySwapper.sol#L335-L337


# Vulnerability details

## Impact
In `StrategySwapper`, swapping from or to tokens with decimals higher than 18 will always revert. This will cause inabilities for strategies to harvest rewards.

## Proof of Concept
[L288](https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/contracts/strategies/StrategySwapper.sol#L288) will revert when `token_` has higher than 18 decimals.
```
 return 10**(18 - IERC20Full(token_).decimals());
```


## Recommended Mitigation Steps
Consider modifying how `_decimalMultiplier` works so it could handle tokens with higher than 18 decimals. 

Update the calculation of `_minTokenAmountOut` and `_minWethAmountOut` to account when decimals are higher/lower than `18`.

