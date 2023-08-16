## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [contracts/Dependencies/CheckContract.sol has a potential gas optimization](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/189) 

# Handle

heiho1


# Vulnerability details

## Impact

CheckContract is used in ActivePool, BorrowerOperations, CollSurplusPool, DefaultPool, HintHelpers, PriceFeed, SortedTroves, StabilityPool, TroveManager, TroveManagerLiquidations, TroveManagerRedemptions but this is a view function and could easily be
implemented as an internal library call.  This would result in slightly larger contract bytecode but should be far more gas efficient than an external contract call as is the current case.

## Proof of Concept

https://medium.com/coinmonks/gas-cost-of-solidity-library-functions-dbe0cedd4678

"""
Use any of the internal calling methods. We prefer internal library calls, because of the associated class features (see Class Features of Solidity by the same author).
Using an external call to a public library function is very expensive, and will only be worth it to avoid including a lot of code into the bytecode for your contract.
Using a local contract component is the most expensive option and should be avoided unless essential.
"""

https://github.com/code-423n4/2021-12-yetifinance/blob/5f5bf61209b722ba568623d8446111b1ea5cb61c/packages/contracts/contracts/Dependencies/CheckContract.sol#L6

## Tools Used

Slither

## Recommended Mitigation Steps

Declare CheckContract as an internal library:

https://medium.com/coinmonks/all-you-should-know-about-libraries-in-solidity-dd8bc953eae7

"""
Embedded Library: If a smart contract is consuming a library which have only internal functions than EVM simply embeds library into the contract. Instead of using delegate call to call a function, it simply uses JUMP statement(normal method call). There is no need to separately deploy library in this scenario.
"""

