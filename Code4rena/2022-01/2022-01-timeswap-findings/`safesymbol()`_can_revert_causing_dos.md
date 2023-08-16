## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [`safeSymbol()` can revert causing DoS](https://github.com/code-423n4/2022-01-timeswap-findings/issues/114) 

# Handle

sirhashalot


# Vulnerability details

## Impact

The `safeSymbol()` function, found in the SafeMetadata.sol contract and called in 4 Timeswap Convenience contracts in the `symbol()` functions, can cause a revert. This could make the 4 contracts not compliant with the ERC20 standard for certain asset pairs, because the `symbol()` function should return a string and not revert.

The root cause of the issue is that the `safeSymbol()` function assumes the return type of any ERC20 token to be a string. If the return value is not a string, abi.decode() will revert, and this will cause the `symbol()` functions in the Timeswap ERC20 contracts to revert.

Because this is known to cause issues with tokens that don't fully follow the ERC20 spec, the `safeSymbol()` function in the BoringCrypto library has a fix for this. The BoringCrypto `safeSymbol()` function is similar to the one in Timeswap but it has a `returnDataToString()` function that handles the case of a bytes32 return value for a token name:
https://github.com/boringcrypto/BoringSolidity/blob/ccb743d4c3363ca37491b87c6c9b24b1f5fa25dc/contracts/libraries/BoringERC20.sol#L15-L39

## Proof of Concept

The root cause is [line 20](https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Convenience/contracts/libraries/SafeMetadata.sol#L20)  of the `safeSymbol()` function in SafeMetadata.sol

The `safeSymbol()` function is called in:
- [Bond.sol](https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Convenience/contracts/Bond.sol#L27-L31)
- [CollateralizedDebt.sol](https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Convenience/contracts/CollateralizedDebt.sol#L38-L42)
- [Insurance.sol](https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Convenience/contracts/Insurance.sol#L29-L33)
- [Liquidity.sol](https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Convenience/contracts/Liquidity.sol#L31-L35)

## Recommended Mitigation Steps

Use the BoringCrypto `safeSymbol()` function code with the `returnDataToString()` parsing function to handle the case of a bytes32 return value:
https://github.com/boringcrypto/BoringSolidity/blob/ccb743d4c3363ca37491b87c6c9b24b1f5fa25dc/contracts/libraries/BoringERC20.sol#L15-L39

