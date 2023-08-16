## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Use immutable variable can save gas](https://github.com/code-423n4/2021-11-yaxis-findings/issues/37) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-11-yaxis/blob/0311dd421fb78f4f174aca034e8239d1e80075fe/contracts/v3/alchemix/adapters/YaxisVaultAdapter.sol#L23-L27

```solidity=23
/// @dev The vault that the adapter is wrapping.
    IVault public vault;

    /// @dev The address which has admin control over this contract.
    address public admin;
```

`vault` and `admin` will never change, use immutable variable instead of storage variable can save gas.

https://github.com/code-423n4/2021-11-yaxis/blob/146febcb61ae7fe20b0920849c4f4bbe111c6ba7/contracts/v3/alchemix/Transmuter.sol#L55-L56

```solidity
address public AlToken;
address public Token;
```

`AlToken` and `Token` can also be changed to `immutable`.

https://github.com/code-423n4/2021-11-yaxis/blob/0311dd421fb78f4f174aca034e8239d1e80075fe/contracts/v3/alchemix/Alchemist.sol#L114-L122

```solidity=114
/// @dev The token that this contract is using as the parent asset.
IMintableERC20 public token;

/// @dev The token that this contract is using as the child asset.
IMintableERC20 public xtoken;
```

`token` and `xtoken` can also be changed to `immutable`.

### Recommendation

Change to:

```solidity
/// @dev The vault that the adapter is wrapping.
IVault public immutable vault;

/// @dev The address which has admin control over this contract.
address public immutable admin;

constructor(IVault _vault, address _admin) public {
    vault = _vault;
    admin = _admin;
    address _token = _vault.getToken();
    IDetailedERC20(_token).safeApprove(address(_vault), uint256(-1));
}
```

