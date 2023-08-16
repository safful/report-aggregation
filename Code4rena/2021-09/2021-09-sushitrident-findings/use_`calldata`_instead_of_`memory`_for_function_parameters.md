## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Use `calldata` instead of `memory` for function parameters](https://github.com/code-423n4/2021-09-sushitrident-findings/issues/119) 

# Handle

hrkrshnn


# Vulnerability details

## Use `calldata` instead of `memory` for function parameters

In some cases, having function arguments in `calldata` instead of
`memory` is more optimal.

Consider the following generic example:

``` solidity
contract C {
    function add(uint[] memory arr) external returns (uint sum) {
        uint length = arr.length;
        for (uint i = 0; i < arr.length; i++) {
            sum += arr[i];
        }
    }
}
```

In the above example, the dynamic array `arr` has the storage location
`memory`. When the function gets called externally, the array values are
kept in `calldata` and copied to `memory` during ABI decoding (using the
opcode `calldataload` and `mstore`). And during the for loop, `arr[i]`
accesses the value in memory using a `mload`. However, for the above
example this is inefficient. Consider the following snippet instead:

``` solidity
contract C {
    function add(uint[] calldata arr) external returns (uint sum) {
        uint length = arr.length;
        for (uint i = 0; i < arr.length; i++) {
            sum += arr[i];
        }
    }
}
```

In the above snippet, instead of going via memory, the value is directly
read from `calldata` using `calldataload`. That is, there are no
intermediate memory operations that carries this value.

**Gas savings**: In the former example, the ABI decoding begins with
copying value from `calldata` to `memory` in a for loop. Each iteration
would cost at least 60 gas. In the latter example, this can be
completely avoided. This will also reduce the number of instructions and
therefore reduces the deploy time cost of the contract.

*In short*, use `calldata` instead of `memory` if the function argument
is only read.

Note that in older Solidity versions, changing some function arguments
from `memory` to `calldata` may cause "unimplemented feature error".
This can be avoided by using a newer (`0.8.*`) Solidity compiler.

### Examples

****Note****: The following pattern is prevalent in the codebase:

``` solidity
function f(bytes memory data) external {
    (...) = abi.decode(data, (..., types, ...));
}
```

Here, changing to `bytes calldata` will decrease the gas. The total
savings for this change across all such uses would be *quite
significant.*

Examples:

``` text
./contracts/examples/PoolFactory.sol:15:    function deployPool(bytes memory _deployData) external override returns (address) {
./contracts/examples/PoolTemplate.sol:12:    constructor(bytes memory _data) {
./contracts/flat/BentoBoxV1Flat.sol:603:    function _getRevertMsg(bytes memory _returnData) internal pure returns (string memory) {
./contracts/pool/HybridPoolFactory.sol:13:    function deployPool(bytes memory _deployData) external returns (address pool) {
./contracts/pool/IndexPoolFactory.sol:13:    function deployPool(bytes memory _deployData) external returns (address pool) {
./contracts/pool/franchised/FranchisedConstantProductPool.sol:54:    constructor(bytes memory _deployData, address _masterDeployer) {
./contracts/pool/franchised/FranchisedHybridPool.sol:60:    constructor(bytes memory _deployData, address _masterDeployer) {
./contracts/pool/franchised/FranchisedIndexPool.sol:61:    constructor(bytes memory _deployData, address _masterDeployer) {
./contracts/pool/HybridPool.sol:60:    constructor(bytes memory _deployData, address _masterDeployer) {
./contracts/pool/ConstantProductPoolFactory.sol:13:    function deployPool(bytes memory _deployData) external returns (address pool) {
./contracts/pool/ConstantProductPool.sol:54:    constructor(bytes memory _deployData, address _masterDeployer) {
./contracts/pool/IndexPool.sol:61:    constructor(bytes memory _deployData, address _masterDeployer) {
./contracts/utils/TridentHelper.sol:143:    function getSelector(bytes memory _data) internal pure returns (bytes4 sig) {
```

Other examples:

1.  <https://github.com/sushiswap/trident/blob/9130b10efaf9c653d74dc7a65bde788ec4b354b5/contracts/TridentRouter.sol#L174>
2.  <https://github.com/sushiswap/trident/blob/9130b10efaf9c653d74dc7a65bde788ec4b354b5/contracts/TridentRouter.sol#L218>


