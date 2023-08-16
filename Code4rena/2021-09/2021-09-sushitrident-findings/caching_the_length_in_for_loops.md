## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Caching the length in for loops](https://github.com/code-423n4/2021-09-sushitrident-findings/issues/112) 

# Handle

hrkrshnn


# Vulnerability details

## Caching the length in for loops

Consider a generic example of an array `arr` and the following loop:

``` solidity
for (uint i = 0; i < arr.length; i++) {
    // do something that doesn't change arr.length
}
```

In the above case, the solidity compiler will always read the length of
the array during each iteration. That is,

1.  if it is a `storage` array, this is an extra `sload` operation (100
    additional extra gas
    ([EIP-2929](https://eips.ethereum.org/EIPS/eip-2929)) for each
    iteration except for the first),
2.  if it is a `memory` array, this is an extra `mload` operation (3
    additional gas for each iteration except for the first),
3.  if it is a `calldata` array, this is an extra `calldataload`
    operation (3 additional gas for each iteration except for the first)

This extra costs can be avoided by caching the array length (in stack):

``` solidity
uint length = arr.length;
for (uint i = 0; i < length; i++) {
    // do something that doesn't change arr.length
}
```

In the above example, the `sload` or `mload` or `calldataload` operation
is only called once and subsequently replaced by a cheap `dupN`
instruction. Even though `mload`, `calldataload` and `dupN` have the
same gas cost, `mload` and `calldataload` needs an additional `pushX
value` to put the offset in the stack, i.e., an extra 3 gas.

This optimization is especially important if it is a storage array or if
it is a lengthy for loop. Note: this especially relevant for the
IndexPool contract.

Note that the Yul based optimizer (not enabled by default; only relevant
if you are using `--experimental-via-ir` or the equivalent in standard
JSON) can sometimes do this caching automatically. However, this is
likely not the case in your project.
[Reference](https://forum.soliditylang.org/t/solidity-team-ama-2-on-wed-10th-of-march-2021/152/15?u=hrkrshnn).

### Examples

``` text
./contracts/flat/BentoBoxV1Flat.sol:626:        for (uint256 i = 0; i < calls.length; i++) {
./contracts/pool/PoolDeployer.sol:34:            for (uint256 i; i < tokens.length - 1; i++) {
./contracts/pool/PoolDeployer.sol:36:                for (uint256 j = i + 1; j < tokens.length; j++) {
./contracts/pool/franchised/FranchisedIndexPool.sol:73:        for (uint256 i = 0; i < _tokens.length; i++) {
./contracts/pool/franchised/FranchisedIndexPool.sol:104:        for (uint256 i = 0; i < tokens.length; i++) {
./contracts/pool/franchised/FranchisedIndexPool.sol:132:        for (uint256 i = 0; i < tokens.length; i++) {
./contracts/pool/franchised/WhiteListManager.sol:105:        for (uint256 i = 0; i < merkleProof.length; i++) {
./contracts/pool/IndexPool.sol:71:        for (uint256 i = 0; i < _tokens.length; i++) {
./contracts/pool/IndexPool.sol:102:        for (uint256 i = 0; i < tokens.length; i++) {
./contracts/pool/IndexPool.sol:129:        for (uint256 i = 0; i < tokens.length; i++) {
./contracts/utils/TridentHelper.sol:27:        for (uint256 i = 0; i < data.length; i++) {
./contracts/TridentRouter.sol:65:        for (uint256 i; i < params.path.length; i++) {
./contracts/TridentRouter.sol:83:        for (uint256 i; i < path.length; i++) {
./contracts/TridentRouter.sol:123:        for (uint256 i; i < params.path.length; i++) {
./contracts/TridentRouter.sol:139:        for (uint256 i; i < params.initialPath.length; i++) {
./contracts/TridentRouter.sol:149:        for (uint256 i; i < params.percentagePath.length; i++) {
./contracts/TridentRouter.sol:157:        for (uint256 i; i < params.output.length; i++) {
./contracts/TridentRouter.sol:181:        for (uint256 i; i < tokenInput.length; i++) {
./contracts/TridentRouter.sol:223:        for (uint256 i; i < minWithdrawals.length; i++) {
./contracts/TridentRouter.sol:225:            for (; j < withdrawnLiquidity.length; j++) {
./contracts/TridentRouter.sol:274:        for (uint256 i; i < tokenInput.length; i++) {
```


