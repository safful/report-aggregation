## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [state variable which can be declared as immutable](https://github.com/code-423n4/2021-06-tracer-findings/issues/40) 

# Handle

JMukesh


# Vulnerability details

## Impact
state variable which have to initialise in constructor can be declared as immutable to save gas

## Proof of Concept

https://docs.soliditylang.org/en/v0.8.4/contracts.html#constant-and-immutable-state-variables

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/Insurance.sol#L23

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/Insurance.sol#L20

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/Liquidation.sol#L30

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/Liquidation.sol#L31

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/Pricing.sol#L17

## Tools Used

manual review

