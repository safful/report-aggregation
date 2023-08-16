## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Revert strings](https://github.com/code-423n4/2021-07-connext-findings/issues/59) 

# Handle

hrkrshnn


# Vulnerability details

## Revert strings

### Consider using custom errors instead of revert strings

Can save gas when the revert condition has been met. And also during
runtime.

### Consider shortening revert strings to less than 32 bytes

Revert strings more than 32 bytes require at least one additional
`mstore`, along with additional operations for computing memory offset,
etc.

Even if you need a string to represent an error, it can usually be done
in less than 32 bytes / characters.

Here are some examples of strings that can be shortened from codebase:

``` txt
./contracts/TransactionManager.sol:96:      "addLiquidity: ETH_WITH_ERC_TRANSFER"
./contracts/TransactionManager.sol:97:      "addLiquidity: ERC20_TRANSFER_FAILED"
./contracts/TransactionManager.sol:122:     "removeLiquidity: INSUFFICIENT_FUNDS"
```

Note that this will only decrease runtime gas when the revert condition
has been met. Regardless, it will decrease deploy time gas.



