## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [Missing setter function for l2MigratorAddr](https://github.com/code-423n4/2022-01-livepeer-findings/issues/167) 

# Handle

defsec


# Vulnerability details

## Impact

Based on the context, l2MigratorAddr should be able to be updated after deployment. However, there is no function to update it. On the L2Migrator.sol, l1MigratorAddr can be updated. (https://github.com/livepeer/arbitrum-lpt-bridge/blob/ebf68d11879c2798c5ec0735411b08d0bea4f287/contracts/L2/gateway/L2Migrator.sol#L101)


## Proof of Concept

1. Navigate to the following contract variable.

https://github.com/livepeer/arbitrum-lpt-bridge/blob/ebf68d11879c2798c5ec0735411b08d0bea4f287/contracts/L1/gateway/L1Migrator.sol#L141

## Tools Used

Code Review

## Recommended Mitigation Steps

Consider to define function for setting l2MigratorAddr.

