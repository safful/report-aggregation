## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Cannot change pending while timelocked](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/30) 

# Handle

joeysantoro


# Vulnerability details

## Impact
If any of the timelocked variables of a basket are pending a change, a transaction to change the target will revert during the timelock window.

## Proof of Concept
Publisher wants to change license fee. They submit a change request but fat finger with the wrong value. The only way to change the pending licenseFee is to complete the change to the incorrect value (after timelock period) then resubmit a new request.

In the case of changing index this can be mitigated by using deleteNewIndex(), however changePublisher and changeLicenseFee cannot be mitigated.

## Recommended Mitigation Steps
Introduce a "setPendingX" method for each of  liscenceFee, publisher, and index. This cleanly separates the logic and allows for overwrite of pending during timelock window.

