## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- resolved

# [Mirrored admin variables in global context, Pool, PoolFactory, Loan and LoanFactory may make it confusing for deployment and maintenance](https://github.com/code-423n4/2021-04-maple-findings/issues/36) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The access control model for the different contracts and how they interact is confusing and may cause issues during deployment and maintenance. Multiple contracts have the notion of admin(s), all of which use setAdmin function to update admin status. This mirroring and reuse of the admin variable is susceptible to accidents.

## Proof of Concept

https://github.com/maple-labs/maple-core/blob/355141befa89c7623150a83b7d56a5f5820819e9/contracts/MapleGlobals.sol#L20
https://github.com/maple-labs/maple-core/blob/355141befa89c7623150a83b7d56a5f5820819e9/contracts/Pool.sol#L55
https://github.com/maple-labs/maple-core/blob/355141befa89c7623150a83b7d56a5f5820819e9/contracts/PoolFactory.sol#L19
https://github.com/maple-labs/maple-core/blob/355141befa89c7623150a83b7d56a5f5820819e9/contracts/Loan.sol#L58
https://github.com/maple-labs/maple-core/blob/355141befa89c7623150a83b7d56a5f5820819e9/contracts/LoanFactory.sol#L27

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Rename the different admin variables e.g. adminGlobal, adminPool, adminLoan. Document the access control roles, hierarchy and interactions explicitly.


