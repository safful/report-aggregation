## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [No need of separate indexing (NFT_ID => Vault Address)](https://github.com/code-423n4/2021-12-mellow-findings/issues/129) 

# Handle

0x421f


# Vulnerability details

If we see vault registry 

https://github.com/code-423n4/2021-12-mellow/blob/6679e2dd118b33481ee81ad013ece4ea723327b5/mellow-vaults/contracts/VaultRegistry.sol#L18

There are two mappings on L18 and L19

On 18 its map(address) => nftID, which is fine 
but one on 19. we can remove, 
Nft IDs are incremental, and are stored in same position in _vaults[] array
so it can be accessed like for id of x => _vaults[x-1] 
with this we dont need to maintain it whenever new vault is deployed, again saving gas cost :)

