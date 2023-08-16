## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed
- PrizePool

# [Preventing zero-address controlled tokens from being added can avoid checks later](https://github.com/code-423n4/2021-06-pooltogether-findings/issues/35) 

# Handle

0xRajeev


# Vulnerability details

## Impact

When _tokenTotalSupply() adds up the supplies of all controlled tokens, it checks and skips zero-address tokens. Instead of checking for zero-address every time for every call to _tokenTotalSupply() from captureAwardBalance() and every deposit via canAddLiquidity modifier, preventing zero-address controlled-token addresses from being added in _addControlledToken() during initialization will avoid these checks.

Impact: All deposit calls which cost 0.5M gas currently will be impacted by these unnecessary checks if we instead perform it one time during the addition of tokens in initialization.

## Proof of Concept

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/PrizePool.sol#L1059

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/PrizePool.sol#L228-L230

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Move zero-address check from time of use to time of adding the tokens into the list in initialize().

