## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Witch can't give back vault after 2x grab](https://github.com/code-423n4/2021-05-yield-findings/issues/8) 

# Handle

gpersoon


# Vulnerability details

## Impact
The witch.sol contract gets access to a vault via the grab function, in case of liquidation.
If the witch.sol contract can't sell the debt within a certain amount of time, a second grab can occur.

After the second grab, the information of the original owner of the vault is lost and the vault can't be returned to the original owner once the debt has been sold.

The grab function stores the previous owner in vaultOwners[vaultId] and then the contract itself is the new owner (via cauldron.grab and cauldron._give).
The vaultOwners[vaultId] is overwritten at the second grab

The function buy of Witch.sol tried to give the vault back to the original owner, which won't succeed after a second grab.

## Proof of Concept
// https://github.com/code-423n4/2021-05-yield/blob/main/contracts/Witch.sol#L50
    function grab(bytes12 vaultId) public {
        DataTypes.Vault memory vault = cauldron.vaults(vaultId);
        vaultOwners[vaultId] = vault.owner;
        cauldron.grab(vaultId, address(this));
    }

// https://github.com/code-423n4/2021-05-yield/blob/main/contracts/Cauldron.sol#L349
    function grab(bytes12 vaultId, address receiver)  external  auth   {
     ...
        _give(vaultId, receiver);
     
// https://github.com/code-423n4/2021-05-yield/blob/main/contracts/Cauldron.sol#L349
 function _give(bytes12 vaultId, address receiver) internal returns(DataTypes.Vault memory vault)  {
    ...
        vault.owner = receiver;
        vaults[vaultId] = vault;

// https://github.com/code-423n4/2021-05-yield/blob/main/contracts/Witch.sol#L57
 function buy(bytes12 vaultId, uint128 art, uint128 min) public { 
    ....
            cauldron.give(vaultId, vaultOwners[vaultId]);

## Tools Used
Editor

## Recommended Mitigation Steps
Assuming it's useful to give back to vault to the original owner:
Make a stack/array of previous owners if multiple instances of the witch.sol contract would be used.
Or check if the witch is already the owner (in the grab function) and keep the vaultOwners[vaultId] if that is the case

