## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- IdleYieldSource

# [User could lose underlying tokens when redeeming from the `IdleYieldSource`](https://github.com/code-423n4/2021-06-pooltogether-findings/issues/120) 

# Handle

shw


# Vulnerability details

## Impact

The `redeemToken` function in `IdleYieldSource` uses `redeemedShare` instead of `redeemAmount` as the input parameter when calling `redeemIdleToken` of the Idle yield source. As a result, users could get fewer underlying tokens than they should.

## Proof of Concept

When burning users' shares, it is correct to use `redeemedShare` (line 130). However, when redeeming underlying tokens from Idle Finance, `redeemAmount` should be used instead of `redeemedShare` (line 131). Usually, the `tokenPriceWithFee()` is greater than `ONE_IDLE_TOKEN`, and thus `redeemedShare` is less than `redeemAmount`, causing users to get fewer underlying tokens than expected.

Referenced code:
[IdleYieldSource.sol#L129-L131](https://github.com/code-423n4/2021-06-pooltogether/blob/main/contracts/yield-source/IdleYieldSource.sol#L129-L131)

## Recommended Mitigation Steps

Change `redeemedShare` to `redeemAmount` at line 131.

