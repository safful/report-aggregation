## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [more efficient gas usage by removing && operator](https://github.com/code-423n4/2022-01-timeswap-findings/issues/89) 

# Handle

rfa


# Vulnerability details

## Impact
more expensive gas usage

## Proof of Concept
instead of using operator && on single require check. using additional require check can save more gas:

https://github.com/code-423n4/2022-01-timeswap/blob/main/Timeswap/Timeswap-V1-Core/contracts/TimeswapPair.sol#L151-L152
https://github.com/code-423n4/2022-01-timeswap/blob/main/Timeswap/Timeswap-V1-Core/contracts/TimeswapPair.sol#L201-L202
https://github.com/code-423n4/2022-01-timeswap/blob/main/Timeswap/Timeswap-V1-Core/contracts/TimeswapPair.sol#L234-L235
https://github.com/code-423n4/2022-01-timeswap/blob/main/Timeswap/Timeswap-V1-Core/contracts/TimeswapPair.sol#L272-L273
https://github.com/code-423n4/2022-01-timeswap/blob/main/Timeswap/Timeswap-V1-Core/contracts/TimeswapPair.sol#L309-L310
https://github.com/code-423n4/2022-01-timeswap/blob/main/Timeswap/Timeswap-V1-Core/contracts/TimeswapPair.sol#L351

## Tools Used
https://remix.ethereum.org

## Recommended Mitigation Steps
example:
require(liquidityTo != address(0), 'E201' );
require(dueTo != address(0), 'E201');


