## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Duplication of Balance](https://github.com/code-423n4/2021-05-yield-findings/issues/16) 

# Handle

0xsomeone


# Vulnerability details

## Impact

It is possible to duplicate currently held `ink` or `art` within a Cauldron, thereby breaking the contract's accounting system minting units out of thin air.

## Proof of Concept

The `stir` function of the `Cauldron`, which can be invoked via a `Ladle` operation, caches balances in memory before decrementing and incrementing. As a result, if a transfer to self is performed, the assignment `balances[to] = balancesTo` will contain the added-to balance instead of the neutral balance.

This allows one to duplicate any number of `ink` or `art` units at will, thereby severely affecting the protocol's integrity. A similar attack was exploited in the third bZx hack resulting in a roughly 8 million loss.

Code Referenced: https://github.com/code-423n4/2021-05-yield/blob/main/contracts/Cauldron.sol#L268-L295

## Tools Used

Manual Review.

## Recommended Mitigation Steps

A `require` check should be imposed that prohibits the `from` and `to` variables to be equivalent.

