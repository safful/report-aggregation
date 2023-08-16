## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- resolved

# [`pendingWithdrawals` not decreased after a `withdraw`](https://github.com/code-423n4/2021-05-fairside-findings/issues/72) 

# Handle

shw


# Vulnerability details

## Impact

The variable `pendingWithdrawals` in the contract `Withdrawable` is not decreased after the function `withdraw` is called, which causes the return value of function `getReserveBalance` less than it should be. This bug could cause incorrect results in several critical functions related to FSD token pricing, including `getFSDPrice`, `purchaseMembership`, `getMaximumBenefitPerUser`, `mint`, and `burn` in the `FSDNetwork` and `FSD` contracts.

## Proof of Concept

Referenced code:
[Withdrawable.sol#L14-L19](https://github.com/code-423n4/2021-05-fairside/blob/main/contracts/dependencies/Withdrawable.sol#L14-L19)
[Withdrawable.sol#L26-L28](https://github.com/code-423n4/2021-05-fairside/blob/main/contracts/dependencies/Withdrawable.sol#L26-L28)

Affected functions:
[FSD.sol#L85](https://github.com/code-423n4/2021-05-fairside/blob/main/contracts/token/FSD.sol#L85)
[FSD.sol#L100](https://github.com/code-423n4/2021-05-fairside/blob/main/contracts/token/FSD.sol#L100)
[FSDNetwork.sol#L136](https://github.com/code-423n4/2021-05-fairside/blob/main/contracts/network/FSDNetwork.sol#L136)
[FSDNetwork.sol#L361](https://github.com/code-423n4/2021-05-fairside/blob/main/contracts/network/FSDNetwork.sol#L361)
[FSDNetwork.sol#L369](https://github.com/code-423n4/2021-05-fairside/blob/main/contracts/network/FSDNetwork.sol#L369)

## Recommended Mitigation Steps

Add `pendingWithdrawals = pendingWithdrawals.sub(reserveAmount);` after line 17 in the contract `Withdrawable`.

