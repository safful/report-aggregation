## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [provideInventory1155 assumes tokenIds.length == amounts.length](https://github.com/code-423n4/2021-12-nftx-findings/issues/117) 

# Handle

sirhashalot


# Vulnerability details

## Impact

The `provideInventory1155()` function in contracts/solidity/NFTXStakingZap.sol contains a for loop that uses tokenIds.length to loop through the amounts array. However, if these two arrays are not the same length, the loop with trigger an error. The error could be triggered after many operations already occur, so checking that these two lengths are equal first could save gas.

## Proof of Concept

The `provideInventory1155()` function contracts/solidity/NFTXStakingZap.sol:
https://github.com/code-423n4/2021-12-nftx/blob/194073f750b7e2c9a886ece34b6382b4f1355f36/nftx-protocol-v2/contracts/solidity/NFTXStakingZap.sol#L201

## Recommended Mitigation Steps

There are two main options to reducing the gas spend in an error condition:
1. Add the following line as the first line of the `provideInventory1155()` function:
`require(tokenIds.length == amounts.length)`
2. In the for loop within the `provideInventory1155()` function, replace `i < tokenIds.length` with `i < amounts.length;`

