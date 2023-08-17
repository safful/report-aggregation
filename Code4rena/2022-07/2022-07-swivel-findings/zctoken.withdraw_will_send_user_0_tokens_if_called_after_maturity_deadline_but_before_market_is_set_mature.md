## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [ZcToken.withdraw will send user 0 tokens if called after maturity deadline but before market is set mature](https://github.com/code-423n4/2022-07-swivel-findings/issues/32) 

# Lines of code

https://github.com/code-423n4/2022-07-swivel/blob/fd36ce96b46943026cb2dfcb76dfa3f884f51c18/Tokens/ZcToken.sol#L99
https://github.com/code-423n4/2022-07-swivel/blob/fd36ce96b46943026cb2dfcb76dfa3f884f51c18/Tokens/ZcToken.sol#L92


# Vulnerability details

## Impact

If `maturityRate` is still `0` after maturity deadline (because no transactions setting `maturityRate` have been executed yet), then `previewWithdraw` calculated amount (used by `ZcToken.withdraw` function) is `0` and thus `withdraw` function will send `0` underlying tokens to user, which might be very confusing to user. Subsequent call to the same function will send him correct amount.

The same problem applies to all view functions in `ZcToken` contract - they use saved market `maturityRate`, which can be `0` even past deadline time and functions revert or return `0` in this case.

Incorrect withdrawal behaviour:

1. Bob has some `ZcToken`s.
2. Right at the time of maturity Bob tries to withdraw his underlying tokens by calling `ZcToken.withdraw` with some underlying amount.
3. Instead of receiving corresponding amount, Bob receives nothing (but transaction still succeeds and he uses gas for it).

## Proof of Concept

1. `withdraw`: calculates `previewAmount` from `previewWithdraw`

https://github.com/code-423n4/2022-07-swivel/blob/fd36ce96b46943026cb2dfcb76dfa3f884f51c18/Tokens/ZcToken.sol#L99


2. `previewWithdraw`: multiplication by `maturityRate` returns 0

https://github.com/code-423n4/2022-07-swivel/blob/fd36ce96b46943026cb2dfcb76dfa3f884f51c18/Tokens/ZcToken.sol#L92

## Recommended Mitigation Steps

Add `getMaturityRate` function to `ZcToken`, which will return either market's `maturityRate` or (if it's `0`) current market's `exchangeRate`. Use this function instead of `maturityRate` everywhere across `ZcToken`.


