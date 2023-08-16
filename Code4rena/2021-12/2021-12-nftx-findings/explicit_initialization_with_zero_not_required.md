## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Explicit initialization with zero not required](https://github.com/code-423n4/2021-12-nftx-findings/issues/207) 

# Handle

Dravee


# Vulnerability details

## Impact
Explicit initialization with zero is not required for variable declaration because uints are 0 by default. Removing this will reduce contract size and save a bit of gas.

## Proof of Concept
Instances include:
```
./NFTXEligibilityManager.sol:85:        for (uint256 i = 0; i < modulesCopy.length; i++) {
./NFTXLPStaking.sol:81:        for (uint256 i = 0; i < vaultIds.length; i++) {
./NFTXLPStaking.sol:206:        for (uint256 i = 0; i < vaultIds.length; i++) {
./NFTXMarketplaceZap.sol:263:    for (uint256 i = 0; i < idsIn.length; i++) {
./NFTXMarketplaceZap.sol:297:    for (uint256 i = 0; i < idsIn.length; i++) {
./NFTXMarketplaceZap.sol:379:    for (uint256 i = 0; i < ids.length; i++) {
./NFTXMarketplaceZap.sol:399:    for (uint256 i = 0; i < ids.length; i++) {
./NFTXMarketplaceZap.sol:414:    for (uint256 i = 0; i < ids.length; i++) {
./NFTXMarketplaceZap.sol:437:    for (uint256 i = 0; i < idsIn.length; i++) {
./NFTXSimpleFeeDistributor.sol:62:    for (uint256 i = 0; i < length; i++) {
 {
./NFTXVaultUpgradeable.sol:364:        for (uint256 i = 0; i < len; i++) {
./NFTXVaultUpgradeable.sol:406:            for (uint256 i = 0; i < tokenIds.length; i++) {
./NFTXVaultUpgradeable.sol:419:            for (uint256 i = 0; i < tokenIds.length; i++) {
./NFTXVaultUpgradeable.sol:442:        for (uint256 i = 0; i < amount; i++) {
```

## Tools Used
Manual Analysis

## Recommended Mitigation Steps
Remove explicit initialization with zero.

