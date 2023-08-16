## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Missing receiver validation in `withdrawFrom`](https://github.com/code-423n4/2022-02-foundation-findings/issues/42) 

# Lines of code

https://github.com/code-423n4/2022-02-foundation/blob/4d8c8931baffae31c7506872bf1100e1598f2754/contracts/FETH.sol#L433


# Vulnerability details

## Impact
The `FETH.withdrawFrom` function does not validate its `to` parameter.
Funds can be lost if `to` is the zero address.

> Similar issues have been judged as medium recently, see [Sandclock M-15](https://code4rena.com/reports/2022-01-sandclock/) / [Github issue](https://github.com/code-423n4/2022-01-sandclock-findings/issues/183#issuecomment-1024626171)

## Recommended Mitigation Steps
Check that `to != 0`.

