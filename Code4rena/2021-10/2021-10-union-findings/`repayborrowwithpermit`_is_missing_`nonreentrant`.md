## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [`repayBorrowWithPermit` is missing `nonReentrant`](https://github.com/code-423n4/2021-10-union-findings/issues/67) 

# Handle

cmichel


# Vulnerability details

The `UToken.repayBorrowWithPermit` function is missing the `repayBorrowWithPermit` modifier which the other repay functions (`repayBorrow`, `repayBorrowBehalf`) have.

## Impact
There's a possibility for re-entrancy. Even though I did not find a way to exploit it, it seems like this function should have the `nonReentrant` modifier as the other similar `repay*` functions have it as well.

## Recommended Mitigation Steps
Add `nonReentrant` to `repayBorrowWithPermit`.

