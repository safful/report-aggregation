## Tags

- bug
- 0 (Non-critical)
- resolved
- sponsor confirmed

# [transferAllowed does not fail](https://github.com/code-423n4/2022-01-openleverage-findings/issues/83) 

# Handle

GeekyLumberjack


# Vulnerability details

## Impact
[transferTokens()](https://github.com/code-423n4/2022-01-openleverage/blob/main/openleverage-contracts/contracts/liquidity/LPool.sol#L95-L135) will not fail when calling [transferAllowed()](https://github.com/code-423n4/2022-01-openleverage/blob/main/openleverage-contracts/contracts/ControllerV1.sol#L88-L91) both [transfer()](https://github.com/code-423n4/2022-01-openleverage/blob/main/openleverage-contracts/contracts/liquidity/LPool.sol#L141) and [transferFrom()](https://github.com/code-423n4/2022-01-openleverage/blob/main/openleverage-contracts/contracts/liquidity/LPool.sol#L150) rely on transferTokens(). Both the name of the function transferAllowed() and the [comments](https://github.com/code-423n4/2022-01-openleverage/blob/main/openleverage-contracts/contracts/liquidity/LPool.sol#L99) above the call show there should be some cases that cause these functions to fail in transferAllowed.


## Tools Used
Manual review

## Recommended Mitigation Steps
Update transfer allowed to include required failures. If there are none, update the comments and the name of the function.

