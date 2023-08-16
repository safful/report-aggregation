## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- Resolved
- disagree with severity

# [Uninitialized Variable `marketWhitelist` in `RCTreasury.sol`](https://github.com/code-423n4/2021-08-realitycards-findings/issues/18) 

# Handle

leastwood


# Vulnerability details

## Impact
The variable, `marketWhitelist`, is never initialized in the contract `RCTreasury.sol`. As a result, the function `marketWhitelistCheck()`  does not perform a proper check on whitelisted users for a restricted market. Additionally, the function will always return `true`, even if a market wishes to restrict its users to a specific role. 

## Proof of Concept

The initial state variable is defined in the link below.
https://github.com/code-423n4/2021-08-realitycards/blob/main/contracts/RCTreasury.sol#L75

The state variable `marketWhitelist` is accessed in the function `RCTreasury.marketWhitelistCheck()` as per below.
https://github.com/code-423n4/2021-08-realitycards/blob/main/contracts/RCTreasury.sol#L269-L281

The function `RCTreasury.marketWhitelistCheck()` is called in `RCMarket.newRental()` as seen below. The comment indicates that there should be some ability to restrict certain markets to specific whitelists, however, there are no methods in `RCTreasury` that allow a market creator to enable this functionality.
https://github.com/code-423n4/2021-08-realitycards/blob/main/contracts/RCMarket.sol#L758-L761

## Tools Used

`npx hardhat coverage`
`slither`
Manual code review

## Recommended Mitigation Steps

Ensure this behaviour is intended. If this is not the case, consider adding a function that enables a market creator to restrict their market to a specific role by whitelisting users.

