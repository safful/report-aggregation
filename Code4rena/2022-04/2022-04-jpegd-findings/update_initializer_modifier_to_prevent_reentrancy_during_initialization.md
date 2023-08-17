## Tags

- bug
- 3 (High Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [Update initializer modifier to prevent reentrancy during initialization](https://github.com/code-423n4/2022-04-jpegd-findings/issues/227) 

# Lines of code

https://github.com/code-423n4/2022-04-jpegd/blob/main/package.json#L18-L19


# Vulnerability details

## Impact

The solution uses:
```jsx
    "@openzeppelin/contracts": "^4.0.0",
    "@openzeppelin/contracts-upgradeable": "^4.3.2",
```
These dependencies have a known high severity vulnerability: 
- https://security.snyk.io/vuln/SNYK-JS-OPENZEPPELINCONTRACTSUPGRADEABLE-2320177
- https://snyk.io/test/npm/@openzeppelin/contracts-upgradeable/4.3.2#SNYK-JS-OPENZEPPELINCONTRACTSUPGRADEABLE-2320177
- https://snyk.io/test/npm/@openzeppelin/contracts/4.0.0#SNYK-JS-OPENZEPPELINCONTRACTS-2320176

Which makes these contracts vulnerable:
```jsx
contracts/helpers/CryptoPunksHelper.sol:
  19:     function initialize(address punksAddress) external initializer {

contracts/helpers/EtherRocksHelper.sol:
  19:     function initialize(address rocksAddress) external initializer {

contracts/staking/JPEGStaking.sol:
  21:     function initialize(IERC20Upgradeable _jpeg) external initializer {

contracts/vaults/FungibleAssetVaultForDAO.sol:
  71:     ) external initializer {

contracts/vaults/NFTVault.sol:
  149:     ) external initializer {
```

## Recommended Mitigation Steps
Upgrade `@openzeppelin/contracts` and `@openzeppelin/contracts-upgradeable` to version 4.4.1 or higher.

