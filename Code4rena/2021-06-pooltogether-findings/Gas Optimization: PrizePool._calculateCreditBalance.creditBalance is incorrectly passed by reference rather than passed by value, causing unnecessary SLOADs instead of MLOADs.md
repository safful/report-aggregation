## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas Optimization: PrizePool._calculateCreditBalance.creditBalance is incorrectly passed by reference rather than passed by value, causing unnecessary SLOADs instead of MLOADs](https://github.com/code-423n4/2021-06-pooltogether-findings/issues/24) 

# Handle

jvaqa


# Vulnerability details

## Impact
PrizePool._calculateCreditBalance.creditBalance is incorrectly declared as storage rather than as memory, causing unnecessary SLOADs instead of MLOADs. [1]

PrizePool._calculateCreditBalance() is declared as a view function, so we know definitively that PrizePool._calculateCreditBalance.creditBalance is not modified within the function. [2]

Since PrizePool._calculateCreditBalance.creditBalance is not modified within the function, then when we fetch it, we want to pass it by value and not by reference by declaring it as 'CreditBalance memory creditBalance' rather than 'CreditBalance storage creditBalance'. 

This way, each of the subsequent reads of the creditBalance are read from memory (MLOAD) rather than read from storage (SLOAD), where MLOAD is cheaper than SLOAD.

## Recommended Mitigation Steps

Change this:

CreditBalance storage creditBalance

To this:

CreditBalance memory creditBalance


[1] https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/PrizePool.sol#L825

[2] https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/PrizePool.sol#L823

