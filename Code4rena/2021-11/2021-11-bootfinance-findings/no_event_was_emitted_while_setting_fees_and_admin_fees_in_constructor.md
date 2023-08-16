## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [No event was emitted while setting fees and admin_fees in constructor](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/128) 

# Handle

JMukesh


# Vulnerability details

## Impact
Event should be emitted after sensitive action like setting fees, admin_fees otherwise it will be difficult track offchain fees changes

## Proof of Concept

https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/core-contracts/contracts/sol/BTCPoolDelegator.sol#L57

https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/core-contracts/contracts/sol/ETHPoolDelegator.sol#L58

https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/core-contracts/contracts/sol/USDPoolDelegator.sol#L53

## Tools Used

manual review
## Recommended Mitigation Steps

event should be emitted after the sensitive action

