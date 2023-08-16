## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Unchecked return value in triggerForToken()](https://github.com/code-423n4/2021-11-nested-findings/issues/76) 

# Handle

palina


# Vulnerability details

## Impact
The Nestedbuybacker::triggerForToken() function does not check the return value of the `ExchangeHelpers.fillQuote(_sellToken, _swapTarget, _swapCallData);` call, which returns a boolean. Even if the swap in the fillQuote() is not successful and no NST was bought, the function proceeds with the trigger() function execution. trigger() also does not check if the `balance` variable (indicating the amount of NST bought) is positive, although there is (at best) no point in executing the rest of the function if there's no NST in the contract.

## Proof of Concept
Unchecked result of the fillQuote() call: https://github.com/code-423n4/2021-11-nested/blob/5d113967cdf7c9ee29802e1ecb176c656386fe9b/contracts/NestedBuybacker.sol#L101
Missing validation in trigger(): https://github.com/code-423n4/2021-11-nested/blob/5d113967cdf7c9ee29802e1ecb176c656386fe9b/contracts/NestedBuybacker.sol#L108

## Tools Used
Manual analysis

## Recommended Mitigation Steps
Add a return value check in the triggerForToken() function:
`bool success = ExchangeHelpers.fillQuote(_sellToken, _swapTarget, _swapCallData);
require(success);`
and/or a `balance` value validation in trigger():
`uint256 balance = NST.balanceOf(address(this));
require(balance > 0);`

