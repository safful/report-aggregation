## Tags

- bug
- QA (Quality Assurance)
- sponsor confirmed

# [Upgrade of Manager.sol inconsistent with interface and other contracts](https://github.com/code-423n4/2022-09-nouns-builder-findings/issues/256) 

# Lines of code

https://github.com/code-423n4/2022-09-nouns-builder/blob/main/src/manager/Manager.sol#L209


# Vulnerability details

### Impact

[Manager.sol](https://github.com/code-423n4/2022-09-nouns-builder/blob/main/src/manager/Manager.sol) implements a different pattern to contract upgradeability only performing an authorisation check and not ensuring that the new Manager implementation has been registered as an upgrade via `isRegisteredUpgrade()`.

The impact is that an upgrade to the [Manager.sol](https://github.com/code-423n4/2022-09-nouns-builder/blob/main/src/manager/Manager.sol) does not require a two step approval and be registered via `registerUpgrade()` . Additionally there is no notification event that the Manager implementation has been registered for an upgrade i.e. `UpgradeRegistered`.

In this respect the [Manager.sol](https://github.com/code-423n4/2022-09-nouns-builder/blob/main/src/manager/Manager.sol) contract has a different implementation to other contracts that make up the DAO (e.g. [Token.sol](https://github.com/code-423n4/2022-09-nouns-builder/blob/main/src/token/Token.sol#L305) and [Governor.sol](https://github.com/code-423n4/2022-09-nouns-builder/blob/main/src/governance/governor/Governor.sol#L618)) and doesn’t follow the process described in the [IManager.sol](https://github.com/code-423n4/2022-09-nouns-builder/blob/main/src/manager/IManager.sol) interface, namely that upgrades are registered via `registerUpgrade()`. and therefore emit the `UpgradeRegistered` event for transparency and monitoring/auditing.

### Proof of Concept

When comparing `_authorizeUpgrade()` in Manager.sol and Token.sol the implementations differ;

```solidity
// Manager.sol
function _authorizeUpgrade(address _newImpl) internal override onlyOwner {}

// Token.sol
function _authorizeUpgrade(address _newImpl) internal view override {
  // Ensure the caller is the shared owner of the token and metadata renderer
  if (msg.sender != owner()) revert ONLY_OWNER();

  // Ensure the implementation is valid
  if (!manager.isRegisteredUpgrade(_getImplementation(), _newImpl)) revert INVALID_UPGRADE(_newImpl);
}
```

When the Manager.sol implementation is updated it **will not** check whether a new implementation has been registered. The `upgradeTo()` function in [UUPS.sol](https://github.com/code-423n4/2022-09-nouns-builder/blob/main/src/lib/proxy/UUPS.sol) will be called checking authorisation and then upgrading the implementation;

```solidity
// UUPS.sol
function upgradeTo(address _newImpl) external onlyProxy {
    _authorizeUpgrade(_newImpl);
    _upgradeToAndCallUUPS(_newImpl, "", false);
}
```

However unlike Token.sol, Manager.sol performs no checks as to whether the implementation has been registered only checking that the calling entity is the owner.

### Tools Used

Vim

### Recommended Remediation Steps

To make Manager.sol consistent with the IManager interface and other contracts in the DAO it should have the same functionality implemented in `_authoriseUpgrade()` (see below);

```solidity
function _authorizeUpgrade(address _newImpl) internal view override onlyOwner {
  if (!this.isRegisteredUpgrade(_getImplementation(), _newImpl)) revert INVALID_UPGRADE(_newImpl);
}
```

As well as this the [NounsBuilderTest.sol](https://github.com/code-423n4/2022-09-nouns-builder/blob/main/test/utils/NounsBuilderTest.sol) should be updated to perform `.registerUpgrade()` before `.upgradeTo()`. For example;

```solidity
// L71 of NounsBuilderTest.sol
vm.startPrank(zoraDAO);
  manager.registerUpgrade(managerImpl0, address(managerImpl));
  manager.upgradeTo(managerImpl);
  vm.stopPrank();
}
```

Then all tests can be run and they will pass.