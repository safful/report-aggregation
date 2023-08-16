## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- YieldSourcePrizePool
- MitigationStarted

# [Actual yield source check on address will succeed for non-existent contract](https://github.com/code-423n4/2021-06-pooltogether-findings/issues/59) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Low-level calls call/delegatecall/staticcall return true even if the account called is non-existent (per EVM design). Solidity documentation warns: "The low-level functions call, delegatecall and staticcall return true as their first return value if the account called is non-existent, as part of the design of the EVM. Account existence must be checked prior to calling if needed.”

The staticcall here will return True even if the _yieldSource contract doesn't exist at any incorrect-but-not-zero address, e.g. EOA address, used during initialization by accident.
Impact: The hack, as commented, to check if it’s an actual yield source contract will fail if the address is indeed a contract account which doesn’t implement the depositToken function. However, if the address is that of an EOA account, the check will pass here but will revert in all future calls to the yield source forcing contract redeployment after the pool is active. Users will not be able to interact with the pool and abandon it.

## Proof of Concept

https://docs.soliditylang.org/en/v0.8.6/control-structures.html#error-handling-assert-require-revert-and-exceptions

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/YieldSourcePrizePool.sol#L41-L45

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

A contract existence check should be performed on _yieldSource prior to the depositToken function existence hack for determining yield source contract.

