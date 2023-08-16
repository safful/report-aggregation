## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [`TokenHandler.transfer` wrong branch order](https://github.com/code-423n4/2021-08-notional-findings/issues/78) 

# Handle

cmichel


# Vulnerability details

## Vulnerability Details
The `TokenHandler.transfer` should handle the `if (token.tokenType == TokenType.Ether)` case first, as if the token type is `Ether` but `netTransferExternal <= 0` it treats the token as an `ERC20` token and tries to call `ERC20` functions on it. 

## Impact
Luckily, trying to call ERC20 functions on the invalid token address will revert which is the desired behavior.

## Recommended Mitigation Steps
We still recommend reordering the branches and adding a `netTransferExternal <= 0` check. The code becomes cleaner and it's more obvious that the transaction will fail.


