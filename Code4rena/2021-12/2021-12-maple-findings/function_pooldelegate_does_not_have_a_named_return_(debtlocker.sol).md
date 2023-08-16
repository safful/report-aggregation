## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Function poolDelegate does not have a named return (DebtLocker.sol)](https://github.com/code-423n4/2021-12-maple-findings/issues/25) 

# Handle

ye0lde


# Vulnerability details

## Impact

Code clarity or possibly gas savings if all the other named returns are in error.

## Proof of Concept

Function `poolDelegate` does not have a named return even though its interface definition does.
The named return isn't used so the fact that it's missing doesn't matter.

Function `pool` and tens of other functions do have a named return. Most of these named returns are not used and could be deleted. I'm assuming this is a project convention and may be used in off-chain reporting, etc.

https://github.com/maple-labs/debt-locker/blob/81f55907db7b23d27e839b9f9f73282184ed4744/contracts/DebtLocker.sol#L279-L285

## Tools Used
Visual Studio Code, Remix

## Recommended Mitigation Steps

Add a named return to function `poolDelegate' for consistency/reporting.

Or if all those other named returns are in error, remove the unused named returns and kick this ticket over to gas optimization.



