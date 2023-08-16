## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Combine external calls into one can save gas](https://github.com/code-423n4/2021-10-slingshot-findings/issues/73) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-10-slingshot/blob/9c0432cca2e43731d5a0ae9c151dacf7835b8719/contracts/Slingshot.sol#L76-L79

```solidity=76
for(uint256 i = 0; i < trades.length; i++) {
    // Checks to make sure that module exists and is correct
    require(moduleRegistry.isModule(trades[i].moduleAddress), "Slingshot: not a module");
}
```

An external call to `moduleRegistry.isModule()` will be called each time in this for loop. They can be combined into one external call by creating an `moduleRegistry.isModuleBatch(address[] memory _moduleAddresses)` function and call that function instead.

### Recommendation

Change to:

```solidity
address[] memory moduleAddresses = new address[](trades.length);
for(uint256 i = 0; i < trades.length; i++) {
    moduleAddresses[i] = trades[i].moduleAddress;
}
require(moduleRegistry.isModuleBatch(moduleAddresses), "Slingshot: not a module");
```

