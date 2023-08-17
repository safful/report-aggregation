## Tags

- bug
- 2 (Med Risk)
- satisfactory
- selected for report
- sponsor confirmed
- M-16

# [MaltRepository._revokeRole may not work correctly](https://github.com/code-423n4/2023-02-malt-findings/issues/5) 

# Lines of code

https://github.com/code-423n4/2023-02-malt/blob/700f9b468f9cf8c9c5cffaa1eba1b8dea40503f9/contracts/Repository.sol#L64-L74


# Vulnerability details

## Impact
MaltRepository inherits from AccessControl and adds validation of validRoles to the hasRole function, which means that even if super.hasRole(role, account) == true, if validRoles[role] == false hasRole will return false, which will cause _revokeRole to not work correctly.
```solidity
  function hasRole(bytes32 role, address account)
    public
    view
    override
    returns (bool)
  {
    // Timelock has all possible permissions
    return
      (super.hasRole(role, account) && validRoles[role]) ||
      super.hasRole(TIMELOCK_ROLE, account);
  }
```
Consider the case where alice is granted ADMIN_ROLE, then ADMIN_ROLE is removed in the removeRole function, validRoles[ADMIN_ROLE] == false. 
```solidity
  function removeRole(bytes32 role) external onlyRole(getRoleAdmin(role)) {
    validRoles[role] = false;
    emit RemoveRole(role);
  }
```
Now if the revokeRole function is called on alice, in the _revokeRole, since hasRole returns false, alice's ADMIN_ROLE will not be revoked.  
Since removeRole ends silently, this may actually cause the caller to incorrectly assume that alice's ADMIN_ROLE has been revoked 
```solidity
    function _revokeRole(bytes32 role, address account) internal virtual {
        if (hasRole(role, account)) {
            _roles[role].members[account] = false;
            emit RoleRevoked(role, account, _msgSender());
        }
    }
```
In addition, the renounceRole and _transferRole functions will also be affected.
In particular, the _transferRole function, if you want to transfer alice's role to bob, both alice and bob will have the role if validRoles[role]==false.
```solidity
  function _transferRole(
    address newAccount,
    address oldAccount,
    bytes32 role
  ) internal {
    _revokeRole(role, oldAccount);
    _grantRole(role, newAccount);
  }
...
    function renounceRole(bytes32 role, address account) public virtual override {
        require(account == _msgSender(), "AccessControl: can only renounce roles for self");

        _revokeRole(role, account);
    }
```
## Proof of Concept
https://github.com/code-423n4/2023-02-malt/blob/700f9b468f9cf8c9c5cffaa1eba1b8dea40503f9/contracts/Repository.sol#L64-L74
https://github.com/code-423n4/2023-02-malt/blob/700f9b468f9cf8c9c5cffaa1eba1b8dea40503f9/contracts/Repository.sol#L99-L102
## Tools Used
None
## Recommended Mitigation Steps
Override renounceRole and removeRole in the MaltRepository and modify them as follows
```diff
    function renounceRole(bytes32 role, address account) public virtual override {
+     require(validRoles[role], "Unknown role");
        require(account == _msgSender(), "AccessControl: can only renounce roles for self");

        _revokeRole(role, account);
    }
...
    function revokeRole(bytes32 role, address account) public virtual override onlyRole(getRoleAdmin(role)) {
+     require(validRoles[role], "Unknown role");
        _revokeRole(role, account);
    }
...
  function _transferRole(
    address newAccount,
    address oldAccount,
    bytes32 role
  ) internal {
+  require(validRoles[role], "Unknown role");
    _revokeRole(role, oldAccount);
    _grantRole(role, newAccount);
  }
```