## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Check condition before calling NestedFactory._handleUnderSpending](https://github.com/code-423n4/2021-11-nested-findings/issues/198) 

# Handle

hyh


# Vulnerability details

## Impact

Whenever condition of the ```_handleUnderSpending``` function fails function call gas costs are wasted. The cost of checking the condition is paid anyway, while when it doesn't hold the function call costs are avoidable.

## Proof of Concept

```_handleUnderSpending``` checks for ```_amountToSpent - _amountSpent > 0```. 
https://github.com/code-423n4/2021-11-nested/blob/main/contracts/NestedFactory.sol#L481

## Recommended Mitigation Steps

When the check condition is false ```_handleUnderSpending``` shouldn't be called and this way the check with corresponding variables to be placed in caller functions:

_submitInOrders
https://github.com/code-423n4/2021-11-nested/blob/main/contracts/NestedFactory.sol#L306

_safeSubmitOrder
https://github.com/code-423n4/2021-11-nested/blob/main/contracts/NestedFactory.sol#L415

