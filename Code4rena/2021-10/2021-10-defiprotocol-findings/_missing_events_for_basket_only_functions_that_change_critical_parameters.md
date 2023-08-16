## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- disagree with severity

# [ Missing events for basket only functions that change critical parameters](https://github.com/code-423n4/2021-10-defiprotocol-findings/issues/81) 

# Handle

defsec


# Vulnerability details

## Impact

Owner only functions that change critical parameters should emit events. Events allow capturing the changed parameters so that off-chain tools/interfaces can register such changes with timelocks that allow users to evaluate them and consider if they would like to engage/exit based on how they perceive the changes as affecting the trustworthiness of the protocol or profitability of the implemented financial services. The alternative of directly querying on-chain contract state for such changes is not considered practical for most users/usages.

Missing events and timelocks do not promote transparency and if such changes immediately affect users’ perception of fairness or trustworthiness, they could exit the protocol causing a reduction in liquidity which could negatively impact protocol TVL and reputation.

There are basket functions that do not emit any events in Auction.sol. 

Missing event : 

https://github.com/code-423n4/2021-10-defiprotocol/blob/7ca848f2779e2e64ed0b4756c02f0137ecd73e50/contracts/contracts/Auction.sol#L44


## Proof of Concept

See similar High-severity H03 finding OpenZeppelin’s Audit of Audius (https://blog.openzeppelin.com/audius-contracts-audit/#high) and Medium-severity M01 finding OpenZeppelin’s Audit of UMA Phase 4 (https://blog.openzeppelin.com/uma-audit-phase-4/)


## Tools Used

## Recommended Mitigation Steps

Add events to all owner/admin functions that change critical parameters. 

