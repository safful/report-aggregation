## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Cache array length in for loops can save gas](https://github.com/code-423n4/2022-01-livepeer-findings/issues/77) 

# Handle

defsec


# Vulnerability details

## Impact

Reading array length at each iteration of the loop takes 6 gas (3 for mload and 3 to place memory_offset) in the stack.

Caching the array length in the stack saves around 3 gas per iteration.

## Proof of Concept

1. Navigate to the following smart contract line.

```
https://github.com/livepeer/arbitrum-lpt-bridge/blob/ebf68d11879c2798c5ec0735411b08d0bea4f287/contracts/L2/gateway/L2Migrator.sol#L197

https://github.com/livepeer/arbitrum-lpt-bridge/blob/ebf68d11879c2798c5ec0735411b08d0bea4f287/contracts/L1/gateway/L1Migrator.sol#L472
```

## Tools Used

None

## Recommended Mitigation Steps

Consider to cache array length.

