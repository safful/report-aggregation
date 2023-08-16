## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [txData.expiry = block.timestamp](https://github.com/code-423n4/2021-07-connext-findings/issues/28) 

# Handle

pauliax


# Vulnerability details

## Impact
function fulfill treats txData.expiry = block.timestamp as expired tx:
    // Make sure the expiry has not elapsed
    require(txData.expiry > block.timestamp, "fulfill: EXPIRED");

However, function cancel has an inclusive check for the same condition:
    if (txData.expiry >= block.timestamp) {
    // Timeout has not expired and tx may only be cancelled by router

## Recommended Mitigation Steps
Unify that to make the code coherent. Probably txData.expiry = block.timestamp should be treated as expired everywhere.

