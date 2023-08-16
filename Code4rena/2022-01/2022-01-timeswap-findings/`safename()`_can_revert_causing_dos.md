## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [`safeName()` can revert causing DoS](https://github.com/code-423n4/2022-01-timeswap-findings/issues/113) 

# Handle

sirhashalot


# Vulnerability details

## Impact

The `safeName()` function, found in the SafeMetadata.sol contract and called in 4 Timeswap Convenience contracts in the `name()` functions, can cause a revert. This could make the 4 contracts not compliant with the ERC20 standard for certain asset pairs, because the `name()` function should return a string and not revert.

The root cause of the issue is that the `safeName()` function assumes the return type of any ERC20 token to be a string. If the return value is not a string, abi.decode() will revert, and this will cause the `name()` functions in the Timeswap ERC20 contracts to revert. There are some tokens that aren't compliant, such as Sai from Maker, which returns a bytes32 value: 
https://kauri.io/#single/dai-token-guide-for-developers/#token-info

Because this is known to cause issues with tokens that don't fully follow the ERC20 spec, the `safeName()` function in the BoringCrypto library has a fix for this. The BoringCrypto `safeName()` function is similar to the one in Timeswap but it has a `returnDataToString()` function that handles the case of a bytes32 return value for a token name:
https://github.com/boringcrypto/BoringSolidity/blob/ccb743d4c3363ca37491b87c6c9b24b1f5fa25dc/contracts/libraries/BoringERC20.sol#L15-L47

## Proof of Concept

The root cause is [line 12](https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Convenience/contracts/libraries/SafeMetadata.sol#L12) of the `safeName()` function in SafeMetadata.sol

The `safeName()` function is called in:
- [Bond.sol](https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Convenience/contracts/Bond.sol#L20-L25)
- [CollateralizedDebt.sol](https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Convenience/contracts/CollateralizedDebt.sol#L22-L36)
- [Insurance.sol](https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Convenience/contracts/Insurance.sol#L20-L27)
- [Liquidity.sol](https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Convenience/contracts/Liquidity.sol#L22-L29)

## Recommended Mitigation Steps

Use the BoringCrypto `safeName()` function code to handle the case of a bytes32 return value:
https://github.com/boringcrypto/BoringSolidity/blob/ccb743d4c3363ca37491b87c6c9b24b1f5fa25dc/contracts/libraries/BoringERC20.sol#L15-L47

