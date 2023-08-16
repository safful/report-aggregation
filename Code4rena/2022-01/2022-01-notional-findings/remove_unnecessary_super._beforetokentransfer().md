## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Remove unnecessary super._beforeTokenTransfer()](https://github.com/code-423n4/2022-01-notional-findings/issues/112) 

# Handle

sirhashalot


# Vulnerability details

## Impact

The sNOTE.sol `_beforeTokenTransfer()` function overrides the ERC20 `_beforeTokenTransfer()` function, but also calls `super._beforeTokenTransfer()`. This call to the parent function is unnecessary because no actions are performed, so it can be removed to save gas. This function call is probably placed here for consistency with the `_afterTokenTransfer()` function, but it is unnecessary with the current code (unlike the call in the `_afterTokenTransfer()` function)

## Proof of Concept

[Line 374 of sNOTE.sol](https://github.com/code-423n4/2022-01-notional/blob/d171cad9e86e0d02e0909eb66d4c24ab6ea6b982/contracts/sNOTE.sol#L374) calls super._beforeTokenTransfer(), which does not need to be called because it performs no actions.

## Recommended Mitigation Steps

Remove line 374 from sNOTE.sol to remove the `super._beforeTokenTransfer()` call 

