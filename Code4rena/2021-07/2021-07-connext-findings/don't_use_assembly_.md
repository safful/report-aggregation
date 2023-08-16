## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [don't use assembly ](https://github.com/code-423n4/2021-07-connext-findings/issues/3) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function revertIfCallFailed of LibUtils.sol uses "assembly" to log error information in a revert situation.

In the latest solidity version this can be done in solidity using the "error" keyword.
See: https://docs.soliditylang.org/en/latest/control-structures.html?#revert
Using pure solidity improves readability.

## Proof of Concept
https://github.com/code-423n4/2021-07-connext/blob/main/contracts/lib/LibUtils.sol#L10
 function revertIfCallFailed(bool success, bytes memory returnData) internal pure {
        if (!success) {
            assembly {  revert(add(returnData, 0x20), mload(returnData))  }
        }
    }

## Tools Used


## Recommended Mitigation Steps
use the error constructs of solidity 0.8.4+



