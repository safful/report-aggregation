## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Unexecutable proposals when Actions.MigrateKernel is not last instruction](https://github.com/code-423n4/2022-08-olympus-findings/issues/51) 

# Lines of code

https://github.com/code-423n4/2022-08-olympus/blob/549b96bcf8b97807738572605f6b1e26b33ef411/src/modules/INSTR.sol#L61


# Vulnerability details

## Impact & Proof Of Concept
In `INSTR.sol`, it is correctly checked that a `ChangeExecutor` instruction only occurs at the last position to avoid situations where the other instructions are deemed as invalid.
However, the same problem can occur for `MigrateKernel`. For instance, let's say we have a `MigrateKernel` followed by a `DeactivatePolicy` action. The `MigrateKernel` action will change the value of `kernel` within the policy. The `DeactivatePolicy` action tries to call `setActiveStatus` on the policy. However, this has a `onlyKernel` modifier and the call will therefore fail when it is done after the value of `kernel` was changed.

## Recommended Mitigation Steps
Perform the same check for `MigrateKernel`.