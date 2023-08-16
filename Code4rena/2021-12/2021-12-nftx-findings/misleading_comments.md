## Tags

- bug
- 0 (Non-critical)
- resolved
- sponsor confirmed

# [Misleading comments](https://github.com/code-423n4/2021-12-nftx-findings/issues/109) 

# Handle

p4st13r4


# Vulnerability details

## Impact

`XTokenUpgradeable.sol` contains many comments regarding SushiBar. It looks like the comments have been copy-pasted from another contract, and may be deceiving for a reader

## Proof of Concept

[https://github.com/code-423n4/2021-12-nftx/blob/main/nftx-protocol-v2/contracts/solidity/token/XTokenUpgradeable.sol#L14](https://github.com/code-423n4/2021-12-nftx/blob/main/nftx-protocol-v2/contracts/solidity/token/XTokenUpgradeable.sol#L14)

## Tools Used

## Recommended Mitigation Steps

Remove said comments

