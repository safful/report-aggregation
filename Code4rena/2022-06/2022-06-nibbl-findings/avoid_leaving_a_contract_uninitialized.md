## Tags

- bug
- QA (Quality Assurance)
- sponsor confirmed

# [Avoid leaving a contract uninitialized](https://github.com/code-423n4/2022-06-nibbl-findings/issues/91) 

# Lines of code

https://github.com/code-423n4/2022-06-nibbl/blob/8c3dbd6adf350f35c58b31723d42117765644110/contracts/NibblVault.sol#L173
https://github.com/code-423n4/2022-06-nibbl/blob/8c3dbd6adf350f35c58b31723d42117765644110/contracts/Basket.sol#L23
https://github.com/code-423n4/2022-06-nibbl/blob/8c3dbd6adf350f35c58b31723d42117765644110/contracts/NibblVaultFactory.sol#L158
https://github.com/code-423n4/2022-06-nibbl/blob/8c3dbd6adf350f35c58b31723d42117765644110/contracts/NibblVaultFactory.sol#L99


# Vulnerability details

## Impact
In OpenZeppelin Contracts (proxy/utils/Initializable.sol):
> CAUTION: An uninitialized contract can be taken over by an attacker. This applies to both a proxy and its implementation...


## Proof of Concept
This can lead to takeover of 2 contracts: `Basket.sol` and `NibblVault.sol` since implementation contracts not initialized and can be initialized publicly.
https://github.com/code-423n4/2022-06-nibbl/blob/8c3dbd6adf350f35c58b31723d42117765644110/contracts/NibblVault.sol

https://github.com/code-423n4/2022-06-nibbl/blob/8c3dbd6adf350f35c58b31723d42117765644110/contracts/Basket.sol


Also, Upgrading either of their implementation in `NibblVaultFactory.sol` when  `proposeNewVaultImplementation(address _newVaultImplementation)` or `proposeNewBasketImplementation(address _newBasketImplementation)` can lead to the same issue if the upgraded contract did not disable initializers.
https://github.com/code-423n4/2022-06-nibbl/blob/8c3dbd6adf350f35c58b31723d42117765644110/contracts/NibblVaultFactory.sol#L158
https://github.com/code-423n4/2022-06-nibbl/blob/8c3dbd6adf350f35c58b31723d42117765644110/contracts/NibblVaultFactory.sol#L99



## Recommended Mitigation Steps
As its mentioned in OpenZeppelin Contracts documentation:

>To prevent the implementation contract from being used, you should invoke the {_disableInitializers} function in the constructor to automatically lock it when it is deployed: 

```
/// @custom:oz-upgrades-unsafe-allow constructor
    constructor() {
        _disableInitializers();
    }
```

Should both of `Basket.sol` and `NibbleVault.sol` use the `_disableInitializers();` which make the implementation contract unable to be initialized to version 1. Hence, for newer version of `Basket.sol` and `NibbleVault.sol` proposed for the factory should also be initialized to version 1 to prevent the attack
https://github.com/code-423n4/2022-06-nibbl/blob/8c3dbd6adf350f35c58b31723d42117765644110/contracts/NibblVaultFactory.sol#L165
https://github.com/code-423n4/2022-06-nibbl/blob/8c3dbd6adf350f35c58b31723d42117765644110/contracts/NibblVaultFactory.sol#L130

