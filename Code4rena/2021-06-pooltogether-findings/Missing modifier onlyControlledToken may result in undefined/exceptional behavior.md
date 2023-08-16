## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- PrizePool

# [Missing modifier onlyControlledToken may result in undefined/exceptional behavior](https://github.com/code-423n4/2021-06-pooltogether-findings/issues/54) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The modifier onlyControlledToken is used for functions that allow the controlledToken address as a parameter to ensure that only whitelisted tokens (ticket and sponsorship) are provided. This is used in all functions except calculateEarlyExitFee().

Impact: The use of a non-whitelisted controlledToken will result in calls to potentially malicious token contract and cause undefined behavior for the `from` user address specified in the call.

## Proof of Concept

Missing modifier:
https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/PrizePool.sol#L729-L747

Modifier:
https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/PrizePool.sol#L1105-L1110

All other functions which accept controlledToken parameter have modifier onlyControlledToken:
https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/PrizePool.sol#L275

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/PrizePool.sol#L299

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/PrizePool.sol#L327

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/PrizePool.sol#L378

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/PrizePool.sol#L418

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/PrizePool.sol#L498

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/PrizePool.sol#L888

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/PrizePool.sol#L903

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Add missing modifier onlyControlledToken to calculateEarlyExitFee().


