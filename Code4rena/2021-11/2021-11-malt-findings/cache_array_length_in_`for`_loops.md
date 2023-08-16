## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Cache array length in `for` loops](https://github.com/code-423n4/2021-11-malt-findings/issues/106) 

# Handle

pmerkleplant


# Vulnerability details

## Impact

Caching the array length in a `for`-loop saves gas as the length does not need
to be read on every iteration.

The following loops could be refactored:
```
./Malt.sol:34:    for (uint256 i = 0; i < minters.length; i = i + 1) {
./Malt.sol:37:    for (uint256 i = 0; i < burners.length; i = i + 1) {
./TransferService.sol:87:    for (uint i = 0; i < verifierList.length - 1; i = i + 1) {
./Auction.sol:407:    for (uint i = 0; i < epochCommitments.length; ++i) {
./libraries/UniswapV2Library.sol:66:        for (uint i; i < path.length - 1; i++) {
./AuctionParticipant.sol:107:    for (uint256 i = replenishingIndex; i < auctionIds.length; i = i + 1) {
./MiningService.sol:49:    for (uint i = 0; i < mines.length; i = i + 1) {
./MiningService.sol:69:    for (uint i = 0; i < mines.length; i = i + 1) {
./MiningService.sol:86:    for (uint i = 0; i < mines.length; i = i + 1) {
./MiningService.sol:96:    for (uint i = 0; i < mines.length; i = i + 1) {
./MiningService.sol:142:    for (uint i = 0; i < mines.length - 1; i = i + 1) {
./MiningService.sol:166:    for (uint i = 0; i < mines.length; i = i + 1) {
./DexHandlers/UniswapHandler.sol:317:    for (uint i = 0; i < buyers.length - 1; i = i + 1) {
```

## Tools used

`grep -rn ".length" .`

