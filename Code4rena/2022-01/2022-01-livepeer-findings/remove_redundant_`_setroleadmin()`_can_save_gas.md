## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Remove redundant `_setRoleAdmin()` can save gas](https://github.com/code-423n4/2022-01-livepeer-findings/issues/196) 

# Handle

WatchPug


# Vulnerability details

https://github.com/livepeer/arbitrum-lpt-bridge/blob/ebf68d11879c2798c5ec0735411b08d0bea4f287/contracts/L2/token/LivepeerToken.sol#L12-L16

```solidity
constructor() ERC20("Livepeer Token", "LPT") ERC20Permit("Livepeer Token") {
    _setupRole(DEFAULT_ADMIN_ROLE, _msgSender());
    _setRoleAdmin(MINTER_ROLE, DEFAULT_ADMIN_ROLE);
    _setRoleAdmin(BURNER_ROLE, DEFAULT_ADMIN_ROLE);
}
```

https://github.com/livepeer/arbitrum-lpt-bridge/blob/ebf68d11879c2798c5ec0735411b08d0bea4f287/contracts/ControlledGateway.sol#L18-L24

```solidity
constructor(address _l1Lpt, address _l2Lpt) {
    _setupRole(DEFAULT_ADMIN_ROLE, _msgSender());
    _setRoleAdmin(GOVERNOR_ROLE, DEFAULT_ADMIN_ROLE);

    l1Lpt = _l1Lpt;
    l2Lpt = _l2Lpt;
}
```

`constant DEFAULT_ADMIN_ROLE = 0x00`

By default, the admin role for all roles is `DEFAULT_ADMIN_ROLE`.

Therefore, `_setRoleAdmin(***_ROLE, DEFAULT_ADMIN_ROLE);` is redundant.

Removing it will make the code simpler and save some gas.

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/783ac759a902a7b4a218c2d026a77e6a26b6c42d/contracts/access/AccessControl.sol#L40-L43

```solidity
 * By default, the admin role for all roles is `DEFAULT_ADMIN_ROLE`, which means
 * that only accounts with this role will be able to grant or revoke other
 * roles. More complex role relationships can be created by using
 * {_setRoleAdmin}.
```

https://docs.openzeppelin.com/contracts/3.x/access-control#granting-and-revoking

> AccessControl includes a special role, called DEFAULT_ADMIN_ROLE, which acts as the ***default admin role for all roles***. An account with this role will be able to manage any other role, unless _setRoleAdmin is used to select a new admin role.

### Recommendation

Remove the redundant code.

