## Tags

- bug
- G (Gas Optimization)
- SwappableYieldSource
- sponsor confirmed

# [Use `abi.encodePacked` for gas optimization](https://github.com/code-423n4/2021-07-pooltogether-findings/issues/53) 

# Handle

shw


# Vulnerability details

## Impact

Changing the `abi.encode` function to `abi.encodePacked` at line 77 of `SwappableYieldSource` can save gas since the `abi.encode` function pads extra null bytes at the end of the call data, which is unnecessary. Also, in general, `abi.encodePacked` is more gas-efficient.

## Proof of Concept

Referenced code:
[SwappableYieldSource.sol#L77](https://github.com/pooltogether/swappable-yield-source/blob/89cf66a3e3f8df24a082e1cd0a0e80d08953049c/contracts/SwappableYieldSource.sol#L77)

[Solidity-Encode-Gas-Comparison](https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison)

## Recommended Mitigation Steps

Change `abi.encode` to `abi.encodePacked` at line 77.

