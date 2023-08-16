## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas optimization on calculating the storage slot of a token](https://github.com/code-423n4/2021-07-sherlock-findings/issues/148) 

# Handle

shw


# Vulnerability details

## Impact

In the `PoolStorage` library, declaring the `POOL_STORAGE_PREFIX` constant with type `bytes32`, and change `abi.encode` ti `abi.encodePacked` at line 87 can save gas.

## Proof of Concept

Referenced code:
[PoolStorage.sol#L14](https://github.com/code-423n4/2021-07-sherlock/blob/main/contracts/storage/PoolStorage.sol#L14)
[PoolStorage.sol#L87](https://github.com/code-423n4/2021-07-sherlock/blob/main/contracts/storage/PoolStorage.sol#L87)

Related links:
[Change `string` to `byteX` if possible](https://medium.com/layerx/how-to-reduce-gas-cost-in-solidity-f2e5321e0395#2a78)
[Solidity-Encode-Gas-Comparison](https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison)

## Recommended Mitigation Steps

See above

