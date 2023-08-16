## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [AMM Cannot Be `initialize()` Except By Governance](https://github.com/code-423n4/2022-02-hubble-findings/issues/51) 

# Lines of code

https://github.com/code-423n4/2022-02-hubble/blob/main/contracts/AMM.sol#L93-L108
https://github.com/code-423n4/2022-02-hubble/blob/main/contracts/AMM.sol#L730-L734
https://github.com/code-423n4/2022-02-hubble/blob/main/contracts/legos/Governable.sol#L10-L13


# Vulnerability details

## Impact

The contact `AMM.sol` cannot be initialize unless it is called from the `_governance` address.

This prevents the use of a deployer account and requires the governance to be able to deploy proxy contracts and encode the required arguements. If this is not feasible then the contract cannot be deployed.

## Proof of Concept

`initialize()` calls `_setGovernace(_governance);` which will store the governance address. 

Following this it will call `syncDeps(_registry);` which has `onlyGovernance` modifier.  Thus, if the `msg.sender` of `initialize()` is not the same as the parameter `_governance` then the initialisation will revert.


```solidity
    function initialize(
        address _registry,
        address _underlyingAsset,
        string memory _name,
        address _vamm,
        address _governance
    ) external initializer {
        _setGovernace(_governance);

        vamm = IVAMM(_vamm);
        underlyingAsset = _underlyingAsset;
        name = _name;
        fundingBufferPeriod = 15 minutes;

        syncDeps(_registry);
    }
```

## Recommended Mitigation Steps

Consider adding the steps manually to `initialize()`. i.e.

```solidity
    function initialize(
        address _registry,
        address _underlyingAsset,
        string memory _name,
        address _vamm,
        address _governance
    ) external initializer {
        _setGovernace(_governance);

        vamm = IVAMM(_vamm);
        underlyingAsset = _underlyingAsset;
        name = _name;
        fundingBufferPeriod = 15 minutes;

        IRegistry registry = IRegistry(_registry);
        clearingHouse = registry.clearingHouse();
        oracle = IOracle(registry.oracle());
}
```

