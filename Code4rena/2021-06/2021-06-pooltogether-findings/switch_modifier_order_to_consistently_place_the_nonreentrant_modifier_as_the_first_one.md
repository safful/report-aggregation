## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Switch modifier order to consistently place the nonreentrant modifier as the first one](https://github.com/code-423n4/2021-06-pooltogether-findings/issues/50) 

# Handle

0xRajeev


# Vulnerability details

## Impact

If a function has multiple modifiers they are executed in the order specified. If checks or logic of modifiers depend on other modifiers this has to be considered in their ordering. PrizePool has functions with multiple modifiers with one of them being nonreentrant which prevents reentrancy on the functions. This should ideally be the first one to prevent even the execution of other modifiers in case of reentrancies.

While there is no obvious vulnerability currently with nonreentrant being the last modifier in the list, it is safer to place it in the first. This is of slight concern with the deposit functions which have the canAddLiquidity() modifier (before nonreentrant) that makes external calls to get totalSupply of controlled tokens.

## Proof of Concept

For reference, see similar finding in Consensys’s audit of Balancer : https://consensys.net/diligence/audits/2020/05/balancer-finance/#switch-modifier-order-in-bpool

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/PrizePool.sol#L275-L277

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/PrizePool.sol#L299-L301


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Switch modifier order to consistently place the nonreentrant modifier as the first one to run so that all other modifiers are executed only if the call is nonreentrant.

