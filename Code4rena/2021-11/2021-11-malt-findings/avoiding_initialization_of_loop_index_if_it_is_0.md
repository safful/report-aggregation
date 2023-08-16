## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Avoiding Initialization of Loop Index If It Is 0](https://github.com/code-423n4/2021-11-malt-findings/issues/116) 

# Handle

Meta0xNull


# Vulnerability details

## Impact
The local variable used as for loop index need not be initialized to 0 because the default value is 0. Avoiding this anti-pattern can save a few opcodes and therefore a tiny bit of gas.

## Proof of Concept
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/TransferService.sol#L87
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/AuctionBurnReserveSkew.sol#L54
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/Malt.sol#L34-L37
More...

## Tools Used
Manual Review

## Recommended Mitigation Steps
Remove explicit 0 initialization of for loop index variable. 

Before:
for (uint i = 0;
After
for (uint i;

