## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- ATokenYieldSource
- IdleYieldSource
- YearnV2YieldSource

# [Missing calls to init functions of inherited contracts](https://github.com/code-423n4/2021-06-pooltogether-findings/issues/60) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Most contracts use the delegateCall proxy pattern and hence their implementations require the use of initialize() functions instead of constructors. This requires derived contracts to call the corresponding init functions of their inherited base contracts. This is done in most places except a few.

Impact: The inherited base classes do not get initialized which may lead to undefined behavior.


## Proof of Concept

Missing call to __ReentrancyGuard_init:
https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/yield-source/ATokenYieldSource.sol#L99-L102

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/yield-source/IdleYieldSource.sol#L59-L61

Missing call to__ERC20_init:
https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/yield-source/IdleYieldSource.sol#L59-L61

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/yield-source/YearnV2YieldSource.sol#L83-L86


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Add missing calls to init functions of inherited contracts.

