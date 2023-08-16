## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed

# [`YaxisVaultAdapter.sol#withdraw()` will most certainly fail](https://github.com/code-423n4/2021-11-yaxis-findings/issues/46) 

# Handle

WatchPug


# Vulnerability details

The actual token withdrawn from `vault.withdraw()` will most certainly less than the `_amount`, due to precision loss in `_tokensToShares()` and `vault.withdraw()`.

As a result, `IDetailedERC20(_token).safeTransfer(_recipient, _amount)` will revert due to insufficant balance.

Based on the simulation we ran, it will fail `99.99%` of the time unless the pps == 1e18.

https://github.com/code-423n4/2021-11-yaxis/blob/146febcb61ae7fe20b0920849c4f4bbe111c6ba7/contracts/v3/alchemix/adapters/YaxisVaultAdapter.sol#L68-L72

```solidity=68
function withdraw(address _recipient, uint256 _amount) external override onlyAdmin {
    vault.withdraw(_tokensToShares(_amount));
    address _token = vault.getToken();
    IDetailedERC20(_token).safeTransfer(_recipient, _amount);
}
```

https://github.com/code-423n4/2021-11-yaxis/blob/146febcb61ae7fe20b0920849c4f4bbe111c6ba7/contracts/v3/Vault.sol#L181-L187

```solidity
function withdraw(
    uint256 _shares
)
    public
    override
{
    uint256 _amount = (balance().mul(_shares)).div(IERC20(address(vaultToken)).totalSupply());
```

### Recommendation

Change to:

```solidity=68
function withdraw(address _recipient, uint256 _amount) external override onlyAdmin {
    address _token = vault.getToken();
    uint256 beforeBalance = IDetailedERC20(_token).balanceOf(address(this));
    
    vault.withdraw(_tokensToShares(_amount));

    IDetailedERC20(_token).safeTransfer(
        _recipient,
        IDetailedERC20(_token).balanceOf(address(this)) - beforeBalance
    );
}
```

