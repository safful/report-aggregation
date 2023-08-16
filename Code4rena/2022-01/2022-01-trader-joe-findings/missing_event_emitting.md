## Tags

- bug
- 0 (Non-critical)
- disagree with severity
- sponsor confirmed

# [Missing event emitting](https://github.com/code-423n4/2022-01-trader-joe-findings/issues/148) 

# Handle

wuwe1


# Vulnerability details

## Impact
Off-chain tools will not work as expected.

## Proof of Concept

Missing UserWithdrawn

[https://github.com/code-423n4/2022-01-trader-joe/blob/main/contracts/LaunchEvent.sol#L132](https://github.com/code-423n4/2022-01-trader-joe/blob/main/contracts/LaunchEvent.sol#L132)

[https://github.com/code-423n4/2022-01-trader-joe/blob/main/contracts/LaunchEvent.sol#L372](https://github.com/code-423n4/2022-01-trader-joe/blob/main/contracts/LaunchEvent.sol#L372)

Missing IssuingTokenDeposited

[https://github.com/code-423n4/2022-01-trader-joe/blob/main/contracts/LaunchEvent.sol#L124](https://github.com/code-423n4/2022-01-trader-joe/blob/main/contracts/LaunchEvent.sol#L124)

[https://github.com/code-423n4/2022-01-trader-joe/blob/main/contracts/LaunchEvent.sol#L287](https://github.com/code-423n4/2022-01-trader-joe/blob/main/contracts/LaunchEvent.sol#L287)


## Recommended Mitigation Steps

Add `emit UserWithdrawn(user, amountMinusFee)` after L372

Add `emit IssuingTokenDeposited(_token, balance)` after L287

