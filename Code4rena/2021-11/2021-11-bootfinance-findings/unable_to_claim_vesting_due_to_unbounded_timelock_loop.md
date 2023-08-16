## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Unable to claim vesting due to unbounded timelock loop](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/120) 

# Handle

nathaniel


# Vulnerability details

## Impact
The timelocks for any *beneficiary* are unbounded, and can be vested by someone who is not the *beneficiary*. When the array becomes significantly big enough, the vestments will no longer be claimable for the *beneficiary*.

The `vest()` function in Vesting.sol does not check the *beneficiary*, hence anyone can vest for anyone else, pushing a new timelock to the `timelocks[_beneficiary]`.
The `_claimableAmount()` function (used by `claim()` function), then loops through the `timelocks[_beneficiary]` to determine the amount to be claimed.
A malicious actor can easy repeatedly call the `vest()` function with minute amounts to make the array large enough, such that when it comes to claiming, it will exceed the gas limit and revert, rendering the vestment for the beneficiary unclaimable. 
The malicious actor could do this to each *beneficiary*, locking up all the vestments.

## Proof of Concept
https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/Vesting.sol#L81
https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/Vesting.sol#L195
https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/Vesting.sol#L148

## Tools Used
Manual code review

## Recommended Mitigation Steps
- Create a minimum on the vestment amounts, such that it won't be feasible for a malicious actor to create a large amount of vestments.
- Restrict the vestment contribution of a *beneficiary* where `require(beneficiary == msg.sender)`

