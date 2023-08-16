## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Style issues](https://github.com/code-423n4/2021-07-connext-findings/issues/29) 

# Handle

pauliax


# Vulnerability details

## Impact
ETHER_ASSETID is a bit missleading name, I think a better name would be NATIVE_ASSETID:
   address constant ETHER_ASSETID = address(0);

Misleading comment (should be 'for fulfillment'):
  // The structure of the signed data for cancellations
  struct SignedFulfillData {

MIN_TIMEOUT could be expressed in days:
 uint256 public constant MIN_TIMEOUT = 1 days; // 24 hours


