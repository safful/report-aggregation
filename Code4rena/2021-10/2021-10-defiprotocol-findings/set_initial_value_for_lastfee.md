## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Set initial value for lastFee](https://github.com/code-423n4/2021-10-defiprotocol-findings/issues/91) 

# Handle

pauliax


# Vulnerability details

## Impact
function handleFees can become cheaper by eliminating this surrounding if/else statement if you initially assign the value to the lastFee upon creation or initialization.

## Recommended Mitigation Steps
  uint256 public override lastFee = block.timestamp;
or in function initialize as it will get this value anyway when doing the initial mintTo. But then you would probably need to skip handleFees if the timeDiff is 0.

