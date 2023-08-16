## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- disagree with severity

# [Missing events for owner only functions that change critical parameters](https://github.com/code-423n4/2021-10-defiprotocol-findings/issues/82) 

# Handle

defsec


# Vulnerability details

## Impact

Owner only functions that change critical parameters should emit events. Events allow capturing the changed parameters so that off-chain tools/interfaces can register such changes with timelocks that allow users to evaluate them and consider if they would like to engage/exit based on how they perceive the changes as affecting the trustworthiness of the protocol or profitability of the implemented financial services. The alternative of directly querying on-chain contract state for such changes is not considered practical for most users/usages.

Missing events and timelocks do not promote transparency and if such changes immediately affect users’ perception of fairness or trustworthiness, they could exit the protocol causing a reduction in liquidity which could negatively impact protocol TVL and reputation.

There are owner functions that do not emit any events in Factory.sol.

Missing event :

https://github.com/code-423n4/2021-10-defiprotocol/blob/main/contracts/contracts/Factory.sol#L39

https://github.com/code-423n4/2021-10-defiprotocol/blob/main/contracts/contracts/Factory.sol#L43

https://github.com/code-423n4/2021-10-defiprotocol/blob/main/contracts/contracts/Factory.sol#L47

https://github.com/code-423n4/2021-10-defiprotocol/blob/main/contracts/contracts/Factory.sol#L51

https://github.com/code-423n4/2021-10-defiprotocol/blob/main/contracts/contracts/Factory.sol#L55
## Proof of Concept

See similar High-severity H03 finding OpenZeppelin’s Audit of Audius (https://blog.openzeppelin.com/audius-contracts-audit/#high) and Medium-severity M01 finding OpenZeppelin’s Audit of UMA Phase 4 (https://blog.openzeppelin.com/uma-audit-phase-4/)


## Tools Used

None

## Recommended Mitigation Steps

Add events to all owner/admin functions that change critical parameters.

