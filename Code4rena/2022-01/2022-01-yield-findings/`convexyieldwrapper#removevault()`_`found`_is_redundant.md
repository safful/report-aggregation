## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [`ConvexYieldWrapper#removeVault()` `found` is redundant](https://github.com/code-423n4/2022-01-yield-findings/issues/111) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2022-01-yield/blob/e946f40239b33812e54fafc700eb2298df1a2579/contracts/ConvexYieldWrapper.sol#L74-L95

```solidity
    function removeVault(bytes12 vaultId, address account) public {
        address owner = cauldron.vaults(vaultId).owner;
        if (account != owner) {
            bytes12[] storage vaults_ = vaults[account];
            uint256 vaultsLength = vaults_.length;
            bool found;
            for (uint256 i = 0; i < vaultsLength; i++) {
                if (vaults_[i] == vaultId) {
                    bool isLast = i == vaultsLength - 1;
                    if (!isLast) {
                        vaults_[i] = vaults_[vaultsLength - 1];
                    }
                    vaults_.pop();
                    found = true;
                    emit VaultRemoved(account, vaultId);
                    break;
                }
            }
            require(found, "Vault not found");
            vaults[account] = vaults_;
        }
    }
```

`found` is redundant, we can just use `return` to stop the whole function when the `vault` to be removed is found and removed.

`removeVault()` can be changed to:

```solidity
function removeVault(bytes12 vaultId, address account) public {
    address owner = cauldron.vaults(vaultId).owner;
    if (account != owner) {
        bytes12[] storage vaults_ = vaults[account];
        uint256 vaultsLength = vaults_.length;
        for (uint256 i = 0; i < vaultsLength; i++) {
            if (vaults_[i] == vaultId) {
                bool isLast = i == vaultsLength - 1;
                if (!isLast) {
                    vaults_[i] = vaults_[vaultsLength - 1];
                }
                vaults_.pop();
                found = true;
                emit VaultRemoved(account, vaultId);
                return;
            }
        }
        revert("Vault not found");
    }
}
```

