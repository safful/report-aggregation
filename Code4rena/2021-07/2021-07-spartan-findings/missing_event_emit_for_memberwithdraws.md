## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- disagree with severity

# [Missing event emit for MemberWithdraws](https://github.com/code-423n4/2021-07-spartan-findings/issues/94) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The DAO member withdrawal is missing an emit for MemberWithdraws event. This results in lack of transparency and off-chain monitoring capability.

## Proof of Concept

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Dao.sol#L78

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Dao.sol#L170-L174


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Add an emit for the event or otherwise rationalize/document why it isn’t necessary and remove the event declaration.

