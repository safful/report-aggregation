## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Unused State variable](https://github.com/code-423n4/2021-06-tracer-findings/issues/38) 

# Handle

JMukesh


# Vulnerability details

## Impact
Unused state variable will increase unnecessarily code size and use the memory

## Proof of Concept

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/oracle/GasOracle.sol#L19

## Tools Used

manual review

## Recommended Mitigation Steps

remove the variable which are unused

