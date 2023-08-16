## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [safeDecimals can revert causing DoS](https://github.com/code-423n4/2022-01-timeswap-findings/issues/112) 

# Handle

sirhashalot


# Vulnerability details

## Impact

The `safeDecimals()` function, found in the SafeMetadata.sol contract and called in 3 different Timeswap Convenience contracts, can cause a revert. This is because the safeDecimals function attempts to use abi.decode to return a uint8 when `data.length >= 32`. However, a data.length value greater than 32 will cause abi.decode to revert.

A similar issue was found in a previoud code4rena contest: https://github.com/code-423n4/2021-05-nftx-findings/issues/46

## Proof of Concept

The root cause is [line 28](https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Convenience/contracts/libraries/SafeMetadata.sol#L28) of the `safeDecimals()` function in SafeMetadata.sol

The following link shows the `safeDecimals()` function in the BoringCrypto library, which might be where this code was borrowed from, uses the strict equality check `data.length == 32`
https://github.com/boringcrypto/BoringSolidity/blob/ccb743d4c3363ca37491b87c6c9b24b1f5fa25dc/contracts/libraries/BoringERC20.sol#L54

`safeDecimals()` is used in multiple functions such as
- CollateralizedDebt.sol [line 50](https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Convenience/contracts/CollateralizedDebt.sol#L50) and [line 54](https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Convenience/contracts/CollateralizedDebt.sol#L54)
- Bond.sol [line 34](https://github.com/code-423n4/2022-01-timeswap/blob/main/Timeswap/Timeswap-V1-Convenience/contracts/Bond.sol#L34)
- Insurance.sol [line 36](https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Convenience/contracts/Insurance.sol#L36)

## Recommended Mitigation Steps

Modify the `safeDecimals()` function to change >= 32 to == 32 like this
`if (success && data.length == 32) return   abi.decode(data, (uint8));`

