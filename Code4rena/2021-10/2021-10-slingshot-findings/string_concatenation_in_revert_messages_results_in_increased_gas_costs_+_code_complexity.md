## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [String concatenation in revert messages results in increased gas costs + code complexity](https://github.com/code-423n4/2021-10-slingshot-findings/issues/37) 

# Handle

TomFrench


# Vulnerability details

## Impact

Switching to custom errors results in reduced deployment/runtime gas cost + ease of decoding revert message

## Proof of Concept

https://github.com/code-423n4/2021-10-slingshot/blob/9c0432cca2e43731d5a0ae9c151dacf7835b8719/contracts/Executioner.sol#L38

Should any of the calls to individual modules fail an error message of the form "<ERROR> Executioner: swap failed: <STEP>" where ERROR is the underlying error message and STEP displays which trade failed

This requires the inclusion of the `ConcatStrings` library and in order to isolate ERROR, knowledge of the string format is necessary. If instead [custom errors](https://blog.soliditylang.org/2021/04/21/custom-errors/) were used, `ConcatStrings` could be removed which results in reduced deployment + runtime costs along with simplifying the codebase. (see ["Errors in Depth"](https://blog.soliditylang.org/2021/04/21/custom-errors/))

```
// old
require(success, appendString(string(data), appendUint(string("Executioner: swap failed: "), i)));

// new
error SwapFailed(uint256 step, bytes errorMessage); // at top of file
if (!success) revert SwapFailed(i, data);
```

If this is done the Executioner's error messages can then be decoded with a standard abi decoder giving greater compatibility with other tools (helpful should you want to filter for certain error strings at some point) without them having to understand the format of your error messages.

Example of a decoded error message with arguments
https://rinkeby.etherscan.io/tx/0x37004044a0a55cce13e2f1dd1813a5f21531cd875fed87ec23ae193e0bb96876

## Recommended Mitigation Steps

Replace `ConcatStrings` library with custom errors.

