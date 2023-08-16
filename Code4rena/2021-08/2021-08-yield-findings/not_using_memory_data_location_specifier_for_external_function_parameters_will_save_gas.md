## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed
- EmergencyBrake
- FYTokenFactory

# [Not using memory data location specifier for external function parameters will save gas](https://github.com/code-423n4/2021-08-yield-findings/issues/45) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Function parameters are passed in calldata. For external functions, these are simply read from calldata. But explicitly specifying memory location for such parameters will force their copying to memory resulting in extra bytecode and more gas. Leaving them in calldata will save gas.

## Proof of Concept

https://github.com/code-423n4/2021-08-yield/blob/4dc46470e616dd0cbd9db9b4742e36c4d809e02c/contracts/utils/EmergencyBrake.sol#L45-L46

https://github.com/code-423n4/2021-08-yield/blob/4dc46470e616dd0cbd9db9b4742e36c4d809e02c/contracts/utils/EmergencyBrake.sol#L66-L67

https://github.com/code-423n4/2021-08-yield/blob/4dc46470e616dd0cbd9db9b4742e36c4d809e02c/contracts/utils/EmergencyBrake.sol#L77-L78

https://github.com/code-423n4/2021-08-yield/blob/4dc46470e616dd0cbd9db9b4742e36c4d809e02c/contracts/utils/EmergencyBrake.sol#L101-L102

https://github.com/code-423n4/2021-08-yield/blob/4dc46470e616dd0cbd9db9b4742e36c4d809e02c/contracts/utils/EmergencyBrake.sol#L116-L117

https://github.com/code-423n4/2021-08-yield/blob/4dc46470e616dd0cbd9db9b4742e36c4d809e02c/contracts/FYTokenFactory.sol#L21-L22

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Do not use memory data location specifier for external function parameters

