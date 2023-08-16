## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Anyone Can Arbitrarily Call `FSDVesting.updateVestedTokens()`](https://github.com/code-423n4/2021-11-fairside-findings/issues/101) 

# Handle

leastwood


# Vulnerability details

## Impact

The `updateVestedTokens()` function is intended to be called by the `FSD.sol` contract when updating a user's vested token amount. A check is performed to ensure that `_user == beneficiary`, however, as `_user` is a user controlled argument, it is possible to spoof calls to `updateVestedTokens()` such that anyone can arbitrarily add any amount to the vested contract. Additionally, there is no check to ensure that the call originated from a trusted/whitelisted source.

There are two main reasons as to why the beneficiary or an attacker would want to call this function:
- To increase the vested amount such that `calculateVestingClaim()` allows them to withdraw their entire vested amount without waiting the entire duration.
- An attacker wishes to block withdrawals from other vested contracts by preventing successful calls to `claimVestedTokens()` by the beneficiary account. This can be done by increasing the vested amount such that `safeTransfer()` calls fail due to insufficient token balance within the contract.

## Proof of Concept

https://github.com/code-423n4/2021-11-fairside/blob/main/contracts/token/FSDVesting.sol#L147-L161
https://github.com/code-423n4/2021-11-fairside/blob/main/contracts/token/FSDVesting.sol#L100-L115
https://github.com/code-423n4/2021-11-fairside/blob/main/contracts/token/FSDVesting.sol#L125
https://github.com/code-423n4/2021-11-fairside/blob/main/contracts/token/FSDVesting.sol#L134

## Tools Used

Manual code review.
Discussions with dev.

## Recommended Mitigation Steps

Ensure that the `updateVestedTokens()` function is only callable from the `FSD.sol` contract. This can be done by implementing an `onlyFSD` role.

