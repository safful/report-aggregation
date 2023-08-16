## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- resolved
- fixed-in-upstream-repo

# [copy paste error in _batchConfirmOutstandingPendingActions](https://github.com/code-423n4/2021-08-floatcapital-findings/issues/5) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function _batchConfirmOutstandingPendingActions of LongShort.sol processeses the variable batched_amountSyntheticToken_toShiftAwayFrom_marketSide,
and sets it to 0 after processing.
However probably due to a copy/paste error, in the second instance, where
batched_amountSyntheticToken_toShiftAwayFrom_marketSide[marketIndex][false] is processed,
the wrong version is set to 0: batched_amountSyntheticToken_toShiftAwayFrom_marketSide[marketIndex][true] = 0

This means the next time the batched_amountSyntheticToken_toShiftAwayFrom_marketSide[marketIndex][false] is processed again.
As it is never reset, it keeps increasing.
The result is that the internal administration will be off and far too many tokens will be shifted tokens from SHORT to LONG.

## Proof of Concept
//https://github.com/code-423n4/2021-08-floatcapital/blob/main/contracts/contracts/LongShort.sol#L1126
LongShort.sol
function _batchConfirmOutstandingPendingActions(
..
    amountForCurrentAction_workingVariable = batched_amountSyntheticToken_toShiftAwayFrom_marketSide[marketIndex][true];
    batched_amountSyntheticToken_toShiftAwayFrom_marketSide[marketIndex][true] = 0;
...    
    amountForCurrentAction_workingVariable = batched_amountSyntheticToken_toShiftAwayFrom_marketSide[marketIndex][false];       
    batched_amountSyntheticToken_toShiftAwayFrom_marketSide[marketIndex][true] = 0; // should probably be false

      
## Tools Used

## Recommended Mitigation Steps
change the second instance of the following (on line 1207)
   batched_amountSyntheticToken_toShiftAwayFrom_marketSide[marketIndex][true] = 0  
to
   batched_amountSyntheticToken_toShiftAwayFrom_marketSide[marketIndex][false] = 0  

p.s. confirmed by Jason of Floatcapital: "Yes, that should definitely be false!"

