## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Members lose SPARTA tokens in removeLiquiditySingle()](https://github.com/code-423n4/2021-07-spartan-findings/issues/133) 

# Handle

0xRajeev


# Vulnerability details

## Impact

When a member calls removeLiquiditySingle() requesting only SPARTA in return, i.e. toBASE = true, the LP tokens are transferred to the Pool to withdraw the constituent SPARTA and TOKENs back to the Router. The withdrawn TOKENs are then transferred back to the Pool to convert to SPARTA and directly transferred to the member from the Pool. However, the member’s SPARTA are left behind in the Router instead of being returned along with converted SPARTA from the Pool. 

In other words, the _member's BASE SPARTA tokens that were removed from the Pool along with the TOKENs are never sent back to the _member because the _token's transferred to the Pool are converted to SPARTA and only those are sent back to member directly from the Pool via swapTo(). 

This effectively results in member losing the SPARTA component of their Pool LP tokens which get left behind in the Router and are possibly claimed by future transactions that remove SPARTA from Router.

## Proof of Concept

LPs sent to Pool: https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Router.sol#L121

SPARTA and TOKENs withdrawn from Pool to Router: https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Router.sol#L122

TOKENs from Router sent to Pool: https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Router.sol#L126

TOKENs in Pool converted to BASE SPARTA and sent to member directly from the Pool: https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Router.sol#L127


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

1. BASE SPARTA should also be transferred to the Pool before swapTo() so they get sent to the member along with the converted TOKENs via swapTo()
2. Use swap(BASE) instead of swapTo() so that TOKENs are swapped for BASE SPARTA in Pool and sent back to ROUTER. Then send all the SPARTA from ROUTER to member.

