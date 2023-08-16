## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- ATokenYieldSource
- IdleYieldSource
- SushiYieldSource
- BadgerYieldSource

# [Lack of `nonReentrant` modifier in yield source contracts](https://github.com/code-423n4/2021-06-pooltogether-findings/issues/119) 

# Handle

shw


# Vulnerability details

## Impact

The `YearnV2YieldSource` contract prevents the `supplyTokenTo`, `redeemToken`, and `sponsor` functions from being reentered by applying a `nonReentrant` modifier. Since these contracts share a similar logic, adding a `nonReentrant` modifier to these functions in all of the yield source contracts is reasonable. However, the same protection is not seen in other yield source contracts.

## Proof of Concept

A `nonReentrant` modifier in the following functions is missing:
1. The `sponsor` function of `ATokenYieldSource`
2. The `supplyTokenTo` and `redeemToken` function of `BadgerYieldSource`
3. The `sponsor` function of `IdleYieldSource`
4. The `supplyTokenTo` and `redeemToken` function of `SushiYieldSource`

Referenced code:
[ATokenYieldSource.sol#L233](https://github.com/code-423n4/2021-06-pooltogether/blob/main/contracts/yield-source/ATokenYieldSource.sol#L233)
[BadgerYieldSource.sol#L43](https://github.com/code-423n4/2021-06-pooltogether/blob/main/contracts/yield-source/BadgerYieldSource.sol#L43)
[BadgerYieldSource.sol#L57](https://github.com/code-423n4/2021-06-pooltogether/blob/main/contracts/yield-source/BadgerYieldSource.sol#L57)
[IdleYieldSource.sol#L150](https://github.com/code-423n4/2021-06-pooltogether/blob/main/contracts/yield-source/IdleYieldSource.sol#L150)
[SushiYieldSource.sol#L47](https://github.com/code-423n4/2021-06-pooltogether/blob/main/contracts/yield-source/SushiYieldSource.sol#L47)
[SushiYieldSource.sol#L66](https://github.com/code-423n4/2021-06-pooltogether/blob/main/contracts/yield-source/SushiYieldSource.sol#L66)

## Recommended Mitigation Steps

Add a `nonReentrant` modifier to these functions. For `BadgerYieldSource` and `SushiYieldSource` contracts, make them inherit from Openzeppelin's `ReentrancyGuardUpgradeable` to use the `nonReentrant` modifier.

