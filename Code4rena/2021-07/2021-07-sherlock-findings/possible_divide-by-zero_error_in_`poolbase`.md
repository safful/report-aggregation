## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Possible divide-by-zero error in `PoolBase`](https://github.com/code-423n4/2021-07-sherlock-findings/issues/136) 

# Handle

shw


# Vulnerability details

## Impact

A possible divide-by-zero error could happen in the `getSherXPerBlock(uint256, IERC20)` function of `PoolBase` when the `totalSupply` of `lockToken` and `_lock` are both 0.

## Proof of Concept

Referenced code:
[PoolBase.sol#L215](https://github.com/code-423n4/2021-07-sherlock/blob/main/contracts/facets/PoolBase.sol#L215)

## Recommended Mitigation Steps

Check if `baseData().lockToken.totalSupply().add(_lock)` equals to 0 before line 214. If so, then return 0.

