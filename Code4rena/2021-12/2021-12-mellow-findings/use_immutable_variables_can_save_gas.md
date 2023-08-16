## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Use immutable variables can save gas](https://github.com/code-423n4/2021-12-mellow-findings/issues/92) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-12-mellow/blob/6679e2dd118b33481ee81ad013ece4ea723327b5/mellow-vaults/contracts/LpIssuer.sol#L20-L20

```solidity=20
IVaultGovernance internal _vaultGovernance;
```

```solidity=35
constructor(
    IVaultGovernance vaultGovernance_,
    address[] memory vaultTokens_,
    string memory name_,
    string memory symbol_
) ERC20(name_, symbol_) {
    require(CommonLibrary.isSortedAndUnique(vaultTokens_), ExceptionsLibrary.SORTED_AND_UNIQUE);
    _vaultGovernance = vaultGovernance_;
    _vaultTokens = vaultTokens_;
    // ...
}
```

`_vaultGovernance` will never change, use immutable variable instead of storage variable can save gas.


https://github.com/code-423n4/2021-12-mellow/blob/6679e2dd118b33481ee81ad013ece4ea723327b5/mellow-vaults/contracts/trader/UniV3Trader.sol#L26-L30

```solidity=26
ISwapRouter public swapRouter;

constructor(address _swapRouter) {
    swapRouter = ISwapRouter(_swapRouter);
}
```

`swapRouter` will never change, use immutable variable instead of storage variable can save gas.

