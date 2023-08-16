## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- disagree with severity

# [Unused _token potentially indicates missing logic or is dead code](https://github.com/code-423n4/2021-07-spartan-findings/issues/128) 

# Handle

0xRajeev


# Vulnerability details

## Impact

_token is conditionally set (to WBNB) but never used in addLiquiditySingleForMember() function unlike its usage in other functions. Such usage typically indicates missing/incorrect functionality. It looks like _handleTransferIn checks token == 0 again to consider BNB.

## Proof of Concept

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Router.sol#L83

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Recommend re-evaluating _token usage in this function, adding any missing logic or removing it for readability/maintainability.

