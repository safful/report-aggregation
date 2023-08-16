## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [Incorrect Natspec can lead to errors](https://github.com/code-423n4/2022-01-insure-findings/issues/176) 

# Handle

0xngndev


# Vulnerability details

## Impact

Unclear Natspec may confuse the user.

In the `fund` function:

- [https://github.com/code-423n4/2022-01-insure/blob/main/contracts/CDSTemplate.sol#L160](https://github.com/code-423n4/2022-01-insure/blob/main/contracts/CDSTemplate.sol#L160)

The Natspec is a copy-paste of the `deposit` function:

- [https://github.com/code-423n4/2022-01-insure/blob/main/contracts/CDSTemplate.sol#L130](https://github.com/code-423n4/2022-01-insure/blob/main/contracts/CDSTemplate.sol#L130)

The problem here is the **receives ITokens** part of the Natspec. The deposit function indeed mints tokens to the `msg.sender` but the `fund` function doesn’t. I would clarify that the `fund` function adds attributions to the surplusPool.

Another minor and unclear bit of Natspec happens here:
[https://github.com/code-423n4/2022-01-insure/blob/main/contracts/Vault.sol#L177](https://github.com/code-423n4/2022-01-insure/blob/main/contracts/Vault.sol#L177)

It describes `_amount` as sender of value instead of something like **amount of value to send.**

## Recommended Mitigation Steps

Explain the Natspec of the `fund` function in more detail. Fix the `transferValue` amount natspec. Also it would be good to add some Natspec to the `defund` function too.

