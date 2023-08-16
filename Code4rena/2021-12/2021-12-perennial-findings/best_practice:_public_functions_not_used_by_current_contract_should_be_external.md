## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Best Practice: public functions not used by current contract should be external](https://github.com/code-423n4/2021-12-perennial-findings/issues/37) 

# Handle

WatchPug


# Vulnerability details

Here are some examples that the code style does not follow the best practices:

https://github.com/code-423n4/2021-12-perennial/blob/fd7c38823833a51ae0c6ae3856a3d93a7309c0e4/protocol/contracts/collateral/Collateral.sol#L171-L173

```solidity=171
function shortfall(IProduct product) public view returns (UFixed18) {
    return _products[product].shortfall;
}
```

https://github.com/code-423n4/2021-12-perennial/blob/fd7c38823833a51ae0c6ae3856a3d93a7309c0e4/protocol/contracts/product/Product.sol#L256-L258

```solidity=256
function maintenance(address account) public view returns (UFixed18) {
    return _positions[account].maintenance(provider);
}
```

https://github.com/code-423n4/2021-12-perennial/blob/fd7c38823833a51ae0c6ae3856a3d93a7309c0e4/protocol/contracts/product/Product.sol#L266-L268

```solidity=266
function maintenanceNext(address account) public view returns (UFixed18) {
    return _positions[account].maintenanceNext(provider);
}
```

https://github.com/code-423n4/2021-12-perennial/blob/fd7c38823833a51ae0c6ae3856a3d93a7309c0e4/protocol/contracts/factory/Factory.sol#L259-L261

```solidity=259
function treasury() public view returns (address) {
    return treasury(0);
}
```

