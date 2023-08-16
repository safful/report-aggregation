## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Pool Credit Line May Not Able to Start When _borrowAsset is Non ERC20 Compliant Tokens](https://github.com/code-423n4/2022-03-sublime-findings/issues/27) 

# Lines of code

https://github.com/sublime-finance/sublime-v1/blob/46536a6d25df4264c1b217bd3232af30355dcb95/contracts/PooledCreditLine/LenderPool.sol#L327


# Vulnerability details

## Impact
```IERC20(_borrowAsset).transfer(_to, _fee);```

If the USDT token is supported as _borrowAsset, the unsafe version of .transfer(_to, _fee) may revert as there is no return value in the USDT token contract’s transfer() implementation (but the IERC20 interface expects a return value).

Function start() will break when _borrowAsset is USDT or Non ERC20 Compliant Tokens. USDT is one of the most borrowed Asset in DEFI. This may cause losing a lot of potential users.

## Proof of Concept
https://github.com/sublime-finance/sublime-v1/blob/46536a6d25df4264c1b217bd3232af30355dcb95/contracts/PooledCreditLine/LenderPool.sol#L327

## Recommended Mitigation Steps
Use .safeTransfer instead of .transfer

```IERC20(_borrowAsset).safeTransfer(_to, _fee);```

