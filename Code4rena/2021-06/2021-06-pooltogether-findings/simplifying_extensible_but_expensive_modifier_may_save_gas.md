## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Simplifying extensible but expensive modifier may save gas](https://github.com/code-423n4/2021-06-pooltogether-findings/issues/27) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The canAddLiquidity modifier, which is used on all deposits (each deposit costs 0.5M gas), appears to be an expensive modifier because it calculates the sum of all the supplies across controlled tokens (by making external CALLs) and adding that up with reserve and timelock supplies. While this is an extensible implementation that supports arbitrary number of controlled tokens via mapped singly linked list, the prize pools typically have only two controlled tokens: tickets and sponsorship. 

Impact: deposits currently cost 0.5M gas.

## Proof of Concept

https://docs.pooltogether.com/protocol/overview#gas-usage

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/PrizePool.sol#L276

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/PrizePool.sol#L300

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/PrizePool.sol#L1119-L1122

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/PrizePool.sol#L1069-L1072

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/PrizePool.sol#L1054-L1064

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Consider gas profiling a fast-path calculation by keeping a separate state variable that tracks the sum of timelock+reserve along with all deposits made towards controlled token supplies and comparing new deposits with that state variable instead of reevaluating totals during each deposit. The extra SLOADs, CALLs and other expensive operations (in linked list and other logic) during reevaluation may add up to more than updating this proposed new state variable across different operations.

