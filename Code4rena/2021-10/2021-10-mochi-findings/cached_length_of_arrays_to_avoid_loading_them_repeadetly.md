## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Cached length of arrays to avoid loading them repeadetly](https://github.com/code-423n4/2021-10-mochi-findings/issues/64) 

# Handle

0x0x0x


# Vulnerability details


## Impact
Gas optimization.

## Proof of Concept

```
for (uint i = 0; i < arr.length; i++) {
    //Operations not effecting the length of the array.
}
```
Loading length for storage arrays cost 100 gas and for memory arrays it costs 3 gas. When arr.length is defined as the condition of for loop, at the start of every iteration the length is loaded from memory. If the length doesn't change during the loop, loading the length of arrays repeatedly can be avoided by saving the length to the stack.
```
uint length = arr.length;
for (uint i = 0; i < length; i++) {
    //Operations not effecting the length of the array.
}
```
By doing so the length is only loaded once rather than loading it as many times as iterations (Therefore, less gas is spent).

## Locations
```
./mochi-core/contracts/profile/MochiProfileV0.sol:68:        for (uint256 i = 0; i < _asset.length; i++) {
./mochi-core/contracts/profile/MochiProfileV0.sol:86:        for (uint256 i = 0; i < _assets.length; i++) {
./mochi-core/contracts/profile/MochiProfileV0.sol:95:        for (uint256 i = 0; i < _assets.length; i++) {
./mochi-cssr/contracts/MochiCSSRv0.sol:41:        for(uint256 i = 0; i<_assets.length; i++){
./mochi-cssr/contracts/MochiCSSRv0.sol:47:        for(uint256 i = 0; i<_assets.length; i++){
./mochi-cssr/contracts/MochiCSSRv0.sol:66:        for(uint256 i = 0; i<_assets.length; i++){
./mochi-cssr/contracts/MochiCSSRv0.sol:77:        for(uint256 i = 0; i<_assets.length; i++){
./mochi-cssr/contracts/adapter/ChainlinkAdapter.sol:34:        for(uint256 i = 0; i<_assets.length; i++) {
./mochi-cssr/contracts/adapter/UniswapV2TokenAdapter.sol:63:        for (uint256 i = 0; i < keyCurrency.length; i++) {
./mochi-cssr/contracts/adapter/UniswapV2TokenAdapter.sol:122:        for (uint256 i = 0; i < keyCurrency.length; i++) {
./mochi-cssr/contracts/adapter/UniswapV2TokenAdapter.sol:175:        for (uint256 i = 0; i < keyCurrency.length; i++) {
./mochi-library/contracts/MerklePatriciaVerifier.sol:36:		for (uint i=0; i<parentNodes.length; i++) {
./mochi-library/contracts/MerklePatriciaVerifier.sol:78:		for(uint i=pathPtr; i<pathPtr+partialPath.length; i++) {
./mochi-library/contracts/MerklePatriciaVerifier.sol:108:		for(uint i=offset; i<nibbleArray.length; i++) {
./mochi-library/contracts/SushiswapV2Library.sol:66:        for (uint i; i < path.length - 1; i++) {
./mochi-library/contracts/SushiswapV2Library.sol:77:        for (uint i = path.length - 1; i > 0; i--) {
./mochi-library/contracts/UniswapV2Library.sol:66:        for (uint i; i < path.length - 1; i++) {
./mochi-library/contracts/UniswapV2Library.sol:77:        for (uint i = path.length - 1; i > 0; i--) {
```
## A similar case

nibblePath.length is constant but it is read at every iteration for require statement.

```./mochi-library/contracts/MerklePatriciaVerifier.sol:36: require(pathPtr <= nibblePath.length, "Path overflow");```



