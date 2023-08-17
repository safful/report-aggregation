## Tags

- bug
- 2 (Med Risk)
- satisfactory
- selected for report
- sponsor confirmed
- edited-by-warden
- M-23

# [StRSR: seizeRSR function fails to update rsrRewardsAtLastPayout variable](https://github.com/code-423n4/2023-01-reserve-findings/issues/64) 

# Lines of code

https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/StRSR.sol#L374-L422
https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/StRSR.sol#L596-L598
https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/StRSR.sol#L496-L530


# Vulnerability details

## Impact
If a RToken is under-collateralized, the `BackingManager` can call the `StRSR.seizeRSR` function ([https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/BackingManager.sol#L141](https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/BackingManager.sol#L141)).  

This sends some amount of `rsr` held by the `StRSR` contract to the `BackingManager` which can then be traded for other tokens in order to recollateralize the RToken.  

There are 3 pools of `rsr` in the `StRSR` contract that `StRSR.seizeRSR` claims `rsr` from.  

1. `stakeRSR` ([https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/StRSR.sol#L386-L398](https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/StRSR.sol#L386-L398))
2. `draftRSR` ([https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/StRSR.sol#L401-L414](https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/StRSR.sol#L401-L414))  
3. rewards ([https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/StRSR.sol#L417](https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/StRSR.sol#L417))  

The `rsr` taken from the rewards is what is interesting in this report.  

The issue is that the `StRSR._payoutRewards` function (which is used to pay `rsr` rewards to stakers over time) keeps track of the available rewards to distribute in the `rsrRewardsAtLastPayout` variable ([https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/StRSR.sol#L517](https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/StRSR.sol#L517)).  

When the `StRSR.seizeRSR` function is called (taking away rewards and sending them to the `BackingManager`) and after that `StRSR._payoutRewards` is called, `StRSR._payoutRewards` uses the `rsrRewardsAtLastPayout` variable that was set before the seizure (the actual amount of rewards is smaller after the seizure).  

Thereby the amount by which `StRSR.stakeRSR` is increased ([https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/StRSR.sol#L513](https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/StRSR.sol#L513)) when rewards are paid out can be greater than the actual rewards that are available.  


## Proof of Concept and further assessment of Impact
The fact that the `rsrRewardsAtLastPayout` variable is too big after a call to `StRSR.seizeRSR` has two consequences when `StRSR._payoutRewards` is called:

1. `stakeRSR` is increased by an amount that is larger than it should be ([https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/StRSR.sol#L513](https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/StRSR.sol#L513))
2. `stakeRate` (which uses division by `stakeRSR` when calculated) is smaller than it should be ([https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/StRSR.sol#L524-L526](https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/StRSR.sol#L524-L526))

Both affected variables can in principle be off by a large amount. In practice this is not likely because the rewards paid out will be small in comparison to `stakeRSR`.  

Also after a second call to `StRSR._payoutRewards` all variables are in sync again and the problem has solved itself. The excess payouts are then accounted for by the `StRSR.rsrRewards` function.  

So there is a small amount of time for any real issue to occur and there does not always occur an issue when `StRSR.seizeRSR` is called.  

That being said, the behavior described so far can cause a temporary DOS:

In `StRSR._payoutRewards`, `stakeRSR` is increased ([https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/StRSR.sol#L513](https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/StRSR.sol#L513)), then `StRSR.rsrRewards` is called which calculates `rsr.balanceOf(address(this)) - stakeRSR - draftRSR` ([https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/StRSR.sol#L596-L598](https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/StRSR.sol#L596-L598)).  

The falsely paid out amount of rewards can increase `StRSR.stakeRSR` so much that this line reverts due to underflow.  

This can cause DOS when `StRSR.seizeRSR` is called again because it internally calls `StRSR.rsrRewards`.  

This will solve itself when more `rsr` accumulates in the contract due to revenue which makes the balance increase or someone can just send `rsr` and thereby increase the balance.  

The DOS occurs also in all functions that internally call `StRSR._payoutRewards` (`StRSR.stake` and `StRSR.unstake`):  

[https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/StRSR.sol#L215](https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/StRSR.sol#L215)  

[https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/StRSR.sol#L262](https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/StRSR.sol#L262)  

Overall the impact of this on the average RToken is quite limited but as explained above it can definitely cause issues.  

## Tools Used
VSCode

## Recommended Mitigation Steps
When `StRSR.seizeRSR` is called, the `rsrRewardsAtLastPayout` variable should be set to the rewards that are available after the seizure:  

```
diff --git a/contracts/p1/StRSR.sol b/contracts/p1/StRSR.sol
index 8fe1c3e7..4f9ea736 100644
--- a/contracts/p1/StRSR.sol
+++ b/contracts/p1/StRSR.sol
@@ -419,6 +419,7 @@ abstract contract StRSRP1 is Initializable, ComponentP1, IStRSR, EIP712Upgradeab
         // Transfer RSR to caller
         emit ExchangeRateSet(initRate, exchangeRate());
         IERC20Upgradeable(address(rsr)).safeTransfer(_msgSender(), seizedRSR);
+        rsrRewardsAtLastPayout = rsrRewards();
     }
```