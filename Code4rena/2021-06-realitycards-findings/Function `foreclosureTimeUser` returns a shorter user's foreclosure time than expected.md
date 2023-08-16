## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- resolved

# [Function `foreclosureTimeUser` returns a shorter user's foreclosure time than expected](https://github.com/code-423n4/2021-06-realitycards-findings/issues/171) 

# Handle

shw


# Vulnerability details

## Impact

The function `foreclosureTimeUser` of `RCTreasury` underestimates the user's foreclosure time if the current time is not the user's last rent calculation time. The underestimation of the foreclosure time could cause wrong results when determining the new owner of the card.

## Proof of Concept

The variable `timeLeftOfDeposit` at line 668 is calculated based on `depositAbleToWithdraw(_user)`, the user's deposit minus the rent from the last rent calculation to the current time. Thus, the variable `timeLeftOfDeposit` indicates the time left of deposit, starting from now. However, at line 672, the `foreclosureTimeWithoutNewCard` is calculated by `timeLeftOfDeposit` plus the user's last rent calculation time instead of the current time. As a result, the user's foreclosure time is reduced. From another perspective, the rent between the last rent calculation time and the current time is counted twice.

Referenced code:
[RCTreasury.sol#L642-L653](https://github.com/code-423n4/2021-06-realitycards/blob/main/contracts/RCTreasury.sol#L642-L653)
[RCTreasury.sol#L669](https://github.com/code-423n4/2021-06-realitycards/blob/main/contracts/RCTreasury.sol#L669)
[RCTreasury.sol#L672](https://github.com/code-423n4/2021-06-realitycards/blob/main/contracts/RCTreasury.sol#L672)
[RCTreasury.sol#L678](https://github.com/code-423n4/2021-06-realitycards/blob/main/contracts/RCTreasury.sol#L678)
[RCOrderbook.sol#L553-L557](https://github.com/code-423n4/2021-06-realitycards/blob/main/contracts/RCOrderbook.sol#L553-L557)

## Recommended Mitigation Steps

Change `depositAbleToWithdraw(_user)` at line 669 to `user[_user].deposit`. Or, change `user[_user].lastRentCalc` at both line 672 and 678 to `block.timestamp`.

