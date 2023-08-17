## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Incorrect implementation of APWine and Tempus `redeem`](https://github.com/code-423n4/2022-06-illuminate-findings/issues/268) 

# Lines of code

https://github.com/code-423n4/2022-06-illuminate/blob/912be2a90ded4a557f121fe565d12ec48d0c4684/redeemer/Redeemer.sol#L136


# Vulnerability details

Redeeming APWine and Tempus PT will always fail, causing a portion of iPT to not be able to be redeemed for the underlying token.

The issue is caused by the incorrect implementation of `redeem`:
```
uint256 amount = IERC20(principal).balanceOf(lender);
Safe.transferFrom(IERC20(u), lender, address(this), amount);
```
The first line correctly calculates the balance of PT token available in `Lender`. However, the second line tries to transfer the underlying token `u` instead of `principal` from Lender to `Redeemer`. Therefore, the redeeming process will always fail as both `APWine.withdraw` and `ITempus.redeemToBacking` will try to redeem non-existent PT.

## Recommended Mitigation Steps
Fix the transfer line:
```
Safe.transferFrom(IERC20(principal), lender, address(this), amount);
```

