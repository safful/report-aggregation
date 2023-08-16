## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Use `calldata` instead of `memory` for function parameters](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/164) 

# Handle

defsec


# Vulnerability details

## Impact

In some cases, having function arguments in calldata instead of
memory is more optimal.

Consider the following generic example:

```
contract C {
function add(uint[] memory arr) external returns (uint sum) {
    uint length = arr.length;
    for (uint i = 0; i < arr.length; i++) {
        sum += arr[i];
    }
}
}
```
In the above example, the dynamic array arr has the storage location
memory. When the function gets called externally, the array values are
kept in calldata and copied to memory during ABI decoding (using the
opcode calldataload and mstore). And during the for loop, arr[i]
accesses the value in memory using a mload. However, for the above
example this is inefficient. Consider the following snippet instead:

```
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
read from calldata using calldataload. That is, there are no
intermediate memory operations that carries this value.

Gas savings: In the former example, the ABI decoding begins with
copying value from calldata to memory in a for loop. Each iteration
would cost at least 60 gas. In the latter example, this can be
completely avoided. This will also reduce the number of instructions and
therefore reduces the deploy time cost of the contract.

In short, use calldata instead of memory if the function argument
is only read.

Note that in older Solidity versions, changing some function arguments
from memory to calldata may cause "unimplemented feature error".
This can be avoided by using a newer (0.8.*) Solidity compiler.

Examples
Note: The following pattern is prevalent in the codebase:

```
function f(bytes memory data) external {
(...) = abi.decode(data, (..., types, ...));
}
```

Here, changing to bytes calldata will decrease the gas. The total
savings for this change across all such uses would be quite
significant.


## Proof Of Concept

Examples:

`https://github.com/code-423n4/2021-12-yetifinance/blob/1da782328ce4067f9654c3594a34014b0329130a/packages/contracts/contracts/BorrowerOperations.sol#L890`

```
https://github.com/code-423n4/2021-12-yetifinance/blob/1da782328ce4067f9654c3594a34014b0329130a/packages/contracts/contracts/BorrowerOperations.sol#L699

https://github.com/code-423n4/2021-12-yetifinance/blob/1da782328ce4067f9654c3594a34014b0329130a/packages/contracts/contracts/BorrowerOperations.sol#L873

https://github.com/code-423n4/2021-12-yetifinance/blob/1da782328ce4067f9654c3594a34014b0329130a/packages/contracts/contracts/BorrowerOperations.sol#L920

https://github.com/code-423n4/2021-12-yetifinance/blob/1da782328ce4067f9654c3594a34014b0329130a/packages/contracts/contracts/BorrowerOperations.sol#L929

https://github.com/code-423n4/2021-12-yetifinance/blob/1da782328ce4067f9654c3594a34014b0329130a/packages/contracts/contracts/ActivePool.sol#L167

```

## Tools Used

None

## Recommended Mitigation Steps

Change memory definition with calldata.
