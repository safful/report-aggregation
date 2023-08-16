## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Checking before external library call can save 2600 gas](https://github.com/code-423n4/2021-07-connext-findings/issues/41) 

# Handle

0xRajeev


# Vulnerability details

## Impact

EIP-2929 in Berlin fork increased the gas costs of CALL* family opcodes to 2600. Making a delegatecall to a library function therefore costs 2600. LibUtils.revertIfCallFailed() reverts and passes on the revert string if the boolean argument is false. Instead, moving the checking of the boolean to the caller avoids the library call when the boolean is true, which is likely the case most of the time.

## Proof of Concept

https://github.com/code-423n4/2021-07-connext/blob/8e1a7ea396d508ed2ebeba4d1898a748255a48d2/contracts/lib/LibUtils.sol#L10-L19

https://github.com/code-423n4/2021-07-connext/blob/8e1a7ea396d508ed2ebeba4d1898a748255a48d2/contracts/lib/LibAsset.sol#L35

https://github.com/code-423n4/2021-07-connext/blob/8e1a7ea396d508ed2ebeba4d1898a748255a48d2/contracts/lib/LibERC20.sol#L20

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Remove the boolean parameter from revertIfCallFailed() and move the conditional check logic to the call sites.

