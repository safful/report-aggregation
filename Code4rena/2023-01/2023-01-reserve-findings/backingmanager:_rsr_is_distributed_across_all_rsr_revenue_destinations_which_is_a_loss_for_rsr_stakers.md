## Tags

- bug
- 2 (Med Risk)
- satisfactory
- selected for report
- sponsor confirmed
- edited-by-warden
- M-12

# [BackingManager: rsr is distributed across all rsr revenue destinations which is a loss for rsr stakers](https://github.com/code-423n4/2023-01-reserve-findings/issues/276) 

# Lines of code

https://github.com/reserve-protocol/protocol/blob/b30ab2068dddf111744b8feed0dd94925e10d947/contracts/p1/BackingManager.sol#L173-L179


# Vulnerability details

## Impact
The `BackingManager.handoutExcessAssets` function sends all `rsr` that the `BackingManager` holds to the `rsrTrader` ([https://github.com/reserve-protocol/protocol/blob/b30ab2068dddf111744b8feed0dd94925e10d947/contracts/p1/BackingManager.sol#L173-L179](https://github.com/reserve-protocol/protocol/blob/b30ab2068dddf111744b8feed0dd94925e10d947/contracts/p1/BackingManager.sol#L173-L179)).  

The purpose of this is that `rsr` which can be held by the `BackingManager` due to seizure from the `StRSR` contract is sent back entirely to the `StRSR` contract and not - as would happen later in the function ([https://github.com/reserve-protocol/protocol/blob/b30ab2068dddf111744b8feed0dd94925e10d947/contracts/p1/BackingManager.sol#L221-L242](https://github.com/reserve-protocol/protocol/blob/b30ab2068dddf111744b8feed0dd94925e10d947/contracts/p1/BackingManager.sol#L221-L242)) - shared across `rsrTrader` and `rTokenTrader`.  

The `rsrTrader` then sends the `rsr` to the `Distributor` ([https://github.com/reserve-protocol/protocol/blob/b30ab2068dddf111744b8feed0dd94925e10d947/contracts/p1/RevenueTrader.sol#L59-L65](https://github.com/reserve-protocol/protocol/blob/b30ab2068dddf111744b8feed0dd94925e10d947/contracts/p1/RevenueTrader.sol#L59-L65)).  

So far so good. However the `Distributor` does not necessarily send all of the `rsr` to the `StRSR` contract. Instead it distributes the `rsr` according to its distribution table. I.e. there can be multiple destinations each receiving a share of the `rsr` ([https://github.com/reserve-protocol/protocol/blob/b30ab2068dddf111744b8feed0dd94925e10d947/contracts/p1/Distributor.sol#L108-L136](https://github.com/reserve-protocol/protocol/blob/b30ab2068dddf111744b8feed0dd94925e10d947/contracts/p1/Distributor.sol#L108-L136)).  

In economic terms, `rsr` that is thereby not sent to `StRSR` but to other destinations, is a transfer of funds from stakers to these destinations, i.e. a loss to stakers.  

Stakers should only pay for recollateralization of the `RToken`, not however send revenue to `rsr` revenue destinations.  

## Proof of Concept
Assume the following situation:  

* A seizure of `rsr` from the `StRSR` contract occurred because the `RToken` was under-collateralized.  

* A trade occurred which restored collateralization. However not all `rsr` was sold by the trade and was returned to the `BackingManager`.  

Now `BackingManager.manageTokens` is called which due to the full collateralization calls `BackingManager.handoutExcessAssets` ([https://github.com/reserve-protocol/protocol/blob/b30ab2068dddf111744b8feed0dd94925e10d947/contracts/p1/BackingManager.sol#L118](https://github.com/reserve-protocol/protocol/blob/b30ab2068dddf111744b8feed0dd94925e10d947/contracts/p1/BackingManager.sol#L118)).  

This sends `rsr` to the `rsrTrader` ([https://github.com/reserve-protocol/protocol/blob/b30ab2068dddf111744b8feed0dd94925e10d947/contracts/p1/BackingManager.sol#L173-L179](https://github.com/reserve-protocol/protocol/blob/b30ab2068dddf111744b8feed0dd94925e10d947/contracts/p1/BackingManager.sol#L173-L179)).  

Then the `rsr` is sent to the `Distributor` ([https://github.com/reserve-protocol/protocol/blob/b30ab2068dddf111744b8feed0dd94925e10d947/contracts/p1/RevenueTrader.sol#L59-L65](https://github.com/reserve-protocol/protocol/blob/b30ab2068dddf111744b8feed0dd94925e10d947/contracts/p1/RevenueTrader.sol#L59-L65)).  

There it is distributed across all `rsr` destinations ([https://github.com/reserve-protocol/protocol/blob/b30ab2068dddf111744b8feed0dd94925e10d947/contracts/p1/Distributor.sol#L108-L136](https://github.com/reserve-protocol/protocol/blob/b30ab2068dddf111744b8feed0dd94925e10d947/contracts/p1/Distributor.sol#L108-L136)).  

## Tools Used
VSCode

## Recommended Mitigation Steps
`rsr` should be sent from the `BackingManager` directly to `StRSR` without the need to go through `rsrTrader` and `Distributor`. Thereby it won't be sent to other `rsr` revenue destinations.  

Fix:  

```
diff --git a/contracts/p1/BackingManager.sol b/contracts/p1/BackingManager.sol
index 431e0796..eb506004 100644
--- a/contracts/p1/BackingManager.sol
+++ b/contracts/p1/BackingManager.sol
@@ -173,7 +173,7 @@ contract BackingManagerP1 is TradingP1, IBackingManager {
         if (rsr.balanceOf(address(this)) > 0) {
             // For CEI, this is an interaction "within our system" even though RSR is already live
             IERC20Upgradeable(address(rsr)).safeTransfer(
-                address(rsrTrader),
+                address(stRSR),
                 rsr.balanceOf(address(this))
             );
         }
```

There is a caveat to this however:  

It is possible for `rsr` to be a reward token for a collateral of the `RToken`.  
Neither the current implementation nor the proposed fix addresses this and instead sends the rewards to `StRSR`.  

In principal, `rsr` that was rewarded should have a share that goes to the `rTokenTrader` as well as include all `rsr` revenue destinations.  
However there is no easy way to differentiate where the `rsr` came from.  

Therefore I think it is reasonable to send all `rsr` to `StRSR` and make it clear to developers and users that `rsr` rewards cannot be paid out to `rToken` holders.  
