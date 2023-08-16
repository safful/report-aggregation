## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- disagree with severity

# [SafeMath not completely used in yield source contracts](https://github.com/code-423n4/2021-06-pooltogether-findings/issues/114) 

# Handle

shw


# Vulnerability details

## Impact

SafeMath is not completely used at the following lines of yield source contracts, which could potentially cause arithmetic underflow and overflow:
1. line 78 in `SushiYieldSource`
2. line 67 in `BadgerYieldSource`
3. line 91 and 98 in `IdleYieldSource`

## Proof of Concept

Referenced code:
[SushiYieldSource.sol#L78](https://github.com/code-423n4/2021-06-pooltogether/blob/main/contracts/yield-source/SushiYieldSource.sol#L78)
[BadgerYieldSource.sol#L67](https://github.com/code-423n4/2021-06-pooltogether/blob/main/contracts/yield-source/BadgerYieldSource.sol#L67)
[IdleYieldSource.sol#L91](https://github.com/code-423n4/2021-06-pooltogether/blob/main/contracts/yield-source/IdleYieldSource.sol#L91)
[IdleYieldSource.sol#L98](https://github.com/code-423n4/2021-06-pooltogether/blob/main/contracts/yield-source/IdleYieldSource.sol#L98)

## Recommended Mitigation Steps

Use the SafeMath library functions in the above lines.

