## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas optimization in IndexTemplate.requestWithdraw()](https://github.com/code-423n4/2022-01-insure-findings/issues/56) 

# Handle

tqts


# Vulnerability details

## Impact
None

## Proof of Concept
In [L197](https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/IndexTemplate.sol#L197) of IndexTemplate, a `_balance` variable is created and initialized to the balance of `msg.sender`. However that variable is used only once in the function.

## Tools Used
Manual review

## Recommended Mitigation Steps
Replace L198 with `require(balanceOf(msg.sender) >= _amount, "ERROR: REQUEST_EXCEED_BALANCE");` and remove L197

