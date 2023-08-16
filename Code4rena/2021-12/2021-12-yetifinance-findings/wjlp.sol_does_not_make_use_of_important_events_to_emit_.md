## Tags

- bug
- 0 (Non-critical)
- disagree with severity
- sponsor confirmed

# [WJLP.sol does not make use of important events to emit ](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/62) 

# Handle

jayjonah8


# Vulnerability details

## Impact
There are no events emitted in the WJLP.sol file for important function calls.  The contract should make use of events for important functions like claimReward() so the protocol can track important events after deployment.  This can help spot unusual activity and assist in monitoring the protocol while its live.  

## Proof of Concept
https://github.com/code-423n4/2021-12-yetifinance/blob/main/packages/contracts/contracts/AssetWrappers/WJLP/WJLP.sol#L215

## Tools Used
Manual code review 



