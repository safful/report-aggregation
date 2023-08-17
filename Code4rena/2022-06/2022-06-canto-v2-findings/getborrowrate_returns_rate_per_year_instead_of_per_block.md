## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [getBorrowRate returns rate per year instead of per block](https://github.com/code-423n4/2022-06-canto-v2-findings/issues/38) 

# Lines of code

https://github.com/Plex-Engineer/lending-market-v2/blob/2646a7676b721db8a7754bf5503dcd712eab2f8a/contracts/NoteInterest.sol#L118
https://github.com/Plex-Engineer/lending-market-v2/blob/2646a7676b721db8a7754bf5503dcd712eab2f8a/contracts/CToken.sol#L209


# Vulnerability details

## Impact
According to the documentation in `InterestRateModel`, `getBorrowRate` has to return the borrow rate per block and the function `borrowRatePerBlock` in `CToken` directly returns the value of `getBorrowRate`. However, the rate per year is returned for `NoteInterest`. Therefore, using `NoteInterest` as an interest model will result in completely wrong values. 

## Recommended Mitigation Steps
Return `baseRatePerBlock`.

