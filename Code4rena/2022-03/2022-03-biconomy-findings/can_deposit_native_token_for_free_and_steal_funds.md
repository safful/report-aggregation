## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Can deposit native token for free and steal funds](https://github.com/code-423n4/2022-03-biconomy-findings/issues/55) 

# Lines of code

https://github.com/code-423n4/2022-03-biconomy/blob/db8a1fdddd02e8cc209a4c73ffbb3de210e4a81a/contracts/hyphen/LiquidityPool.sol#L151


# Vulnerability details

## Impact
The `depositErc20` function allows setting `tokenAddress = NATIVE` and does not throw an error.
No matter the `amount` chosen, the `SafeERC20Upgradeable.safeTransferFrom(IERC20Upgradeable(tokenAddress), sender, address(this), amount);` call will not revert because it performs a low-level call to `NATIVE = 0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE`, which is an EOA, and the low-level calls to EOAs always succeed.
Because the `safe*` version is used, the EOA not returning any data does not revert either.

This allows an attacker to deposit infinite native tokens by not paying anything.
The contract will emit the same `Deposit` event as a real `depositNative` call and the attacker receives the native funds on the other chain.

## Recommended Mitigation Steps
Check `tokenAddress != NATIVE` in `depositErc20`.


