## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Caching weth in timeswapMintCallback can save gas](https://github.com/code-423n4/2022-01-timeswap-findings/issues/107) 

# Handle

p4st13r4


# Vulnerability details

## Impact

In `TimeswapConvenience.sol` the `weth` state variable is read twice. It can just be immediately assigned locally so that the two `deposit` calls avoid reading the same variable from storage

## Proof of Concept

- [https://github.com/code-423n4/2022-01-timeswap/blob/main/Timeswap/Timeswap-V1-Convenience/contracts/TimeswapConvenience.sol#L505](https://github.com/code-423n4/2022-01-timeswap/blob/main/Timeswap/Timeswap-V1-Convenience/contracts/TimeswapConvenience.sol#L505)
- [https://github.com/code-423n4/2022-01-timeswap/blob/main/Timeswap/Timeswap-V1-Convenience/contracts/TimeswapConvenience.sol#L512](https://github.com/code-423n4/2022-01-timeswap/blob/main/Timeswap/Timeswap-V1-Convenience/contracts/TimeswapConvenience.sol#L512)

## Tools Used

Editor

## Recommended Mitigation Steps

Assign `weth` to `localWeth`

