## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Argument order for SavingsAccount approval functions is odd](https://github.com/code-423n4/2021-12-sublime-findings/issues/76) 

# Handle

TomFrenchBlockchain


# Vulnerability details

## Impact

Possible confusion

## Proof of Concept

https://github.com/code-423n4/2021-12-sublime/blob/9df1b7c4247f8631647c7627a8da9bdc16db8b11/contracts/SavingsAccount/SavingsAccount.sol#L326-L368

Compare this to the standard ERC20 versions.

```
approve(address spender, uint256 amount)

vs 

approve(uint256 amount, address token, address to)

```

Having the amount at the beginning is very odd imo and I'd expect it at the end.

## Recommended Mitigation Steps

Change `approve(uint256 amount, address token, address to)` to `approve(address token, address to, uint256 amount)` and similar for other functions.

I'd also change `to` to the standard `spender` but this is nbd.




