## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [No message in require statements.](https://github.com/code-423n4/2021-11-malt-findings/issues/98) 

# Handle

BouSalman


# Vulnerability details

## Vulnerability description
There is multiple instances within the **Malt** protocol codebase that do not append messages to the require statements.

## Impact
add a custom message to the require statement to create a better sense of what's is the reason of failure.

## Proof of Concept
https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/Auction.sol#L681
https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/ERC20Permit.sol#L56
https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/ERC20Permit.sol#L76
https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/ERC20Permit.sol#L78
https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/ERC20Permit.sol#L119

## Tools Used
manual code review.

## Recommended Mitigation Steps
append custom message to the require statements.

