## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [function which can declared as external ](https://github.com/code-423n4/2021-06-tracer-findings/issues/62) 

# Handle

JMukesh


# Vulnerability details

## Impact
public function which are not called within contract should be declared as external
to save gas

## Proof of Concept

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/TracerPerpetualSwaps.sol#L572

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/Liquidation.sol#L470

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/InsurancePoolToken.sol#L14

## Tools Used

manual review

## Recommended Mitigation Steps

Declare public function as external which are not called in the contract

