## Tags

- bug
- duplicate
- 2 (Med Risk)
- sponsor confirmed

# [Undercollateralized vaults' owner can be overwritten](https://github.com/code-423n4/2021-05-yield-findings/issues/30) 

# Handle

cmichel


# Vulnerability details

The witch can `Witch.grab` vaults and the `vaultOwners[vaultId]` field is set to the original owner.
However, when the auction time is over and the debt has not been fully paid back, the original owner is not restored, and the witch can grab the same vault again, overwriting the original owner `vaultOwners[vaultId]` field permanently with the witch.

```solidity
function grab(bytes12 vaultId) public {
    DataTypes.Vault memory vault = cauldron.vaults(vaultId);
    vaultOwners[vaultId] = vault.owner;
    cauldron.grab(vaultId, address(this));
}
```

Even a full repayment will not restore the original vault owner anymore.

## Impact
No funds will be stuck as the vault can still be correctly liquidated (calling `settle`).
However, the vault owner will not be restored which is bad if it is a valuable vaultId (low number) that has a special meaning or would be used as an NFT/for retroactive airdrops for initial liquidity providers down the road.

## Recommended Mitigation Steps
When grabbing check if `vaultOwners[vaultId]` is already the witch and in that case just do an early return of the function - not overwriting the `vaultOwners[vaultId]` field.


