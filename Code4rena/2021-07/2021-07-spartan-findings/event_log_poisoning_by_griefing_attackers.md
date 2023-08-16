## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Event log poisoning by griefing attackers](https://github.com/code-423n4/2021-07-spartan-findings/issues/104) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Event log poisoning is possible by griefing attackers who have no DAO weight but vote and emit event that takes up event log space.

## Proof of Concept

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Dao.sol#L382

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Dao.sol#L393


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Emit event only if non-zero weight as relevant to proposal voting/cancelling.

