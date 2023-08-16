## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [`NestedFactory.unlockTokens` fails to use safe transfer](https://github.com/code-423n4/2021-11-nested-findings/issues/78) 

# Handle

elprofesor


# Vulnerability details

## Impact
The use of `_token.transfer()` in `NestedFactory.unlockTokens` may have unintended consequences. ERC20 tokens can implement contra to the EIP20 spec (USDT for instance returns VOID). This may result in tokens that return anything from false to void and these return values would not throw on failure. As a result transfer's in `unlockTokens` may not appropriately throw on failure.

## Proof of Concept
https://github.com/code-423n4/2021-11-nested/blob/5d113967cdf7c9ee29802e1ecb176c656386fe9b/contracts/NestedFactory.sol#L271-L273

## Recommended Mitigation Steps
We recommend using OpenZeppelin’s SafeERC20 versions with the `safeTransfer` function that handles the return value check as well as non-standard-compliant tokens.

