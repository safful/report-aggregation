## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- disagree with severity

# [Dangerous use of storage data location specifier](https://github.com/code-423n4/2021-06-tracer-findings/issues/61) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Reference type local variables require an explicit data location specified indicating if they need to be in storage or memory. Assigning state variables to storage local variables creates a reference  (instead of a copy) to the state variable and modifications to the local variable will be reflected in the state variable. This is required if the intention is to make updates to state variables. Unnecessarily using storage specifiers may lead to unintentional updates of state variables and has led to vulnerabilities.

In L457 of settle(), a local variable insuranceBalance is created in storage to point to balances[address(insuranceContract)] but is never updated. Instead balances[address(insuranceContract)] itself is updated on L474.

Impact: While there is no immediate impact, any modifications to the code with insuranceBalance will be dangerous because it will update the critical state variable balances[address(insuranceContract)]. It is safer to use a memory specifier for insuranceBalance.

## Proof of Concept

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/TracerPerpetualSwaps.sol#L456-L457

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/TracerPerpetualSwaps.sol#L467

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/TracerPerpetualSwaps.sol#L474

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Replace the use of storage specifier on L457 with memory.

