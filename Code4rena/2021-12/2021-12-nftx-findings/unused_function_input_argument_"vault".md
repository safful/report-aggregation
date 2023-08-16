## Tags

- bug
- 0 (Non-critical)
- resolved
- sponsor confirmed

# [Unused function input argument "vault"](https://github.com/code-423n4/2021-12-nftx-findings/issues/205) 

# Handle

PPrieditis


# Vulnerability details

## Impact
NFTXMarketplaceZap.sol function _buyVaultToken() has unused parameter "vault" 
https://github.com/code-423n4/2021-12-nftx/blob/main/nftx-protocol-v2/contracts/solidity/NFTXMarketplaceZap.sol#L497

## Recommended Mitigation Steps
Remove parameter "vault" from _buyVaultToken() and update necessary _buyVaultToken() calls.

