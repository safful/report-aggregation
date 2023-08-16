## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [NFTXInventoryStaking._deployXToken create2 deploy result isn't zero checked](https://github.com/code-423n4/2021-12-nftx-findings/issues/115) 

# Handle

hyh


# Vulnerability details

## Impact

deployXTokenForVault call will not revert on deploy failure.

## Proof of Concept

NFTXInventoryStaking._deployXToken is called by deployXTokenForVault:
https://github.com/code-423n4/2021-12-nftx/blob/main/nftx-protocol-v2/contracts/solidity/NFTXInventoryStaking.sol#L64

_deployXToken uses deploy generated address without check:
https://github.com/code-423n4/2021-12-nftx/blob/main/nftx-protocol-v2/contracts/solidity/NFTXInventoryStaking.sol#L160

## Recommended Mitigation Steps

Require non zero deployedXToken address before calling __XToken_init.

