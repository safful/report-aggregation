## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [`ConvexYieldWrapper.sol` Redundant code](https://github.com/code-423n4/2022-01-yield-findings/issues/107) 

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

At L77, `vaults_` is defined as `vaults[account]`, thus `vaults[account] = vaults_` at L93 is redundant.

https://github.com/code-423n4/2022-01-yield/blob/e946f40239b33812e54fafc700eb2298df1a2579/contracts/ConvexYieldWrapper.sol#L57-L69

```solidity
function addVault(bytes12 vaultId) external {
    address account = cauldron.vaults(vaultId).owner;
    require(account != address(0), "No owner for the vault");
    bytes12[] storage vaults_ = vaults[account];
    uint256 vaultsLength = vaults_.length;

    for (uint256 i = 0; i < vaultsLength; i++) {
        require(vaults_[i] != vaultId, "Vault already added");
    }
    vaults_.push(vaultId);
    vaults[account] = vaults_;
    emit VaultAdded(account, vaultId);
}
```

Similarly, L76 is redundant.

