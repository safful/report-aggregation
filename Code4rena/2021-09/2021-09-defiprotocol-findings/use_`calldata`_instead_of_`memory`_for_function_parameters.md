## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Use `calldata` instead of `memory` for function parameters](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/229) 

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

### Examples

1.  <https://github.com/code-423n4/2021-09-defiProtocol/blob/main/contracts/contracts/Factory.sol#L65>
2.  <https://github.com/code-423n4/2021-09-defiProtocol/blob/main/contracts/contracts/Auction.sol#L69>
3.  <https://github.com/code-423n4/2021-09-defiProtocol/blob/main/contracts/contracts/Auction.sol#L140>
4.  <https://github.com/code-423n4/2021-09-defiProtocol/blob/main/contracts/contracts/Basket.sol#L36>
5.  <https://github.com/code-423n4/2021-09-defiProtocol/blob/main/contracts/contracts/Basket.sol#L53>
6.  <https://github.com/code-423n4/2021-09-defiProtocol/blob/main/contracts/contracts/Basket.sol#L170>


