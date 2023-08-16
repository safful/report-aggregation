## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [`++i` costs less gass compared to `i++`](https://github.com/code-423n4/2021-12-nftx-findings/issues/195) 

# Handle

Dravee


# Vulnerability details

## Impact
`++i` costs less gass compared to `i++` for unsigned integer, as pre-increment is cheaper (about 5 gas per iteration)

## Proof of Concept
`i++` increments `i` and returns the initial value of `i`. Which means:

```
uint i = 1;
i++; // == 1 but i == 2
```

But `++i` returns the actual incremented value:

```
uint i = 1;
++i; // == 2 and i == 2 too, so no need for a temporary variable
```

In the first case, the compiler has to create a temporary variable (when used) for returning `1` instead of `2`

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
./NFTXStakingZap.sol:192:    for (uint256 i = 0; i < tokenIds.length; i++) {
./NFTXStakingZap.sol:203:    for (uint256 i = 0; i < tokenIds.length; i++) {
./NFTXStakingZap.sol:341:    for (uint256 i = 0; i < ids.length; i++) {
./NFTXVaultUpgradeable.sol:267:            for (uint256 i = 0; i < tokenIds.length; i++) {
./NFTXVaultUpgradeable.sol:406:            for (uint256 i = 0; i < tokenIds.length; i++) {
./NFTXVaultUpgradeable.sol:419:            for (uint256 i = 0; i < tokenIds.length; i++) {
```

## Tools Used
VS Code

## Recommended Mitigation Steps
Use `++i` instead of `i++` to increment the value of an uint variable

