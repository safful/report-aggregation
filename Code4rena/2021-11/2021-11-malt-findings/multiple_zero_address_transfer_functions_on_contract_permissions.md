## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Multiple Zero address transfer functions on contract Permissions](https://github.com/code-423n4/2021-11-malt-findings/issues/64) 

# Handle

BouSalman


# Vulnerability details

## Vulnerability description
On contract **Permissions.sol** there is multiple functions to withdraws funds, these functions currently do not check for zero value address before doing the transaction.

## Impact
Loss of funds, ETHs and ERC20.

## Proof of Concept
https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/Permissions.sol#L80
https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/Permissions.sol#L88
https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/Permissions.sol#L97
https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/Permissions.sol#L104

## Tools Used
manual code review.

## Recommended Mitigation Steps
use require() statement to validate address address(0) before sending the funds.

