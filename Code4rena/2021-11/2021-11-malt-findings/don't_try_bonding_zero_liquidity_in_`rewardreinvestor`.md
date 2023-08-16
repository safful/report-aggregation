## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Don't try bonding zero liquidity in `RewardReinvestor`](https://github.com/code-423n4/2021-11-malt-findings/issues/101) 

# Handle

pmerkleplant


# Vulnerability details

## Impact

Function `RewardReinvestor::_bondAccount` tries to bond liquidity to an account,
even though it is known whether the liquidity is zero.

## Proof of Concept

The return value `liquidityCreated` in [line 105](https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/RewardReinvestor.sol#L105) can be zero.
The following function call, `bondToAccount()`, then reverts with "Cannot bond 0".

## Recommended Mitigation Steps

Gas could be saved if the function would revert earlier, i.e. in [line 106](https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/RewardReinvestor.sol#L106),
if the `liquidityCreated` is zero.

