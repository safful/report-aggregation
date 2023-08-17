## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- edited-by-warden
- M-25

# [BackingManager: rTokens might not be redeemable when protocol is paused due to missing token allowance](https://github.com/code-423n4/2023-01-reserve-findings/issues/16) 

# Lines of code

https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/RToken.sol#L439-L514
https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/BackingManager.sol#L72-L77


# Vulnerability details

## Impact
The Reserve protocol allows redemption of rToken even when the protocol is `paused`.  

The `docs/system-design.md` documentation describes the `paused` state as:  

>all interactions disabled EXCEPT RToken.redeem + RToken.cancel + ERC20 functions + StRSR.stake

Redemption of rToken should only ever be prohibited when the protocol is in the `frozen` state.  

The issue is that the `RToken.redeem` function ([https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/RToken.sol#L439-L514](https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/RToken.sol#L439-L514)) relies on the `BackingManager.grantRTokenAllowance` function ([https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/BackingManager.sol#L72-L77](https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/BackingManager.sol#L72-L77)) to be called before redemption.  

Also the only function that relies on `BackingManager.grantRTokenAllowance` to be called before is `RToken.redeem`.  

Therefore `BackingManager.grantRTokenAllowance` can be called at any time before a specific ERC20 needs first be transferred from the `BackingManager` for the purpose of redemption of rToken.  

The issue is that the `BackingManager.grantRTokenAllowance` function has the `notPausedOrFrozen` modifier. This means it cannot (in contrast to `RToken.redeem`) be called when the protocol is `paused`.  

Therefore if rToken is for the first time redeemed for a specific ERC20 in a `paused` protocol state, `BackingManager.grantRTokenAllowance` might not have been called before.  

This effectively disables redemption of rToken as long as the protocol is `paused` and is clearly against the usability / economic considerations to allow redemption in the `paused` state.  

## Proof of Concept
For simplicity assume there is an rToken backed by a single ERC20 called AToken

1. rToken is issued and AToken is transferred to the `BackingManager`.
2. The protocol goes into the `paused` state before any redemptions have occurred. So the `BackingManager.grantRTokenAllowance` function might not have been called at this point.
3. Now the protocol is `paused` which should allow redemption of rToken but it is not possible because the AToken allowance cannot be granted since the `BackingManager.grantRTokenAllowance` function cannot be called in the `paused` state.

Another scenario is when the basket of a RToken is changed to include an ERC20 that was not included in the basket before. If the protocol now goes into the `paused` state without `BackingManager.grantRTokenAllowance` being called before, redemption is not possible.  

## Tools Used
VSCode

## Recommended Mitigation Steps
The `BackingManager.grantRTokenAllowance` function should use the `notFrozen` modifier instead of the `notPausedOrFrozen` modifier such that allowance can be granted in the `paused` state:  

```
diff --git a/contracts/p1/BackingManager.sol b/contracts/p1/BackingManager.sol
index 431e0796..7dfa29e9 100644
--- a/contracts/p1/BackingManager.sol
+++ b/contracts/p1/BackingManager.sol
@@ -69,7 +69,7 @@ contract BackingManagerP1 is TradingP1, IBackingManager {
     // checks: erc20 in assetRegistry
     // action: set allowance on erc20 for rToken to UINT_MAX
     // Using two safeApprove calls instead of safeIncreaseAllowance to support USDT
-    function grantRTokenAllowance(IERC20 erc20) external notPausedOrFrozen {
+    function grantRTokenAllowance(IERC20 erc20) external notFrozen {
         require(assetRegistry.isRegistered(erc20), "erc20 unregistered");
         // == Interaction ==
         IERC20Upgradeable(address(erc20)).safeApprove(address(main.rToken()), 0);
```