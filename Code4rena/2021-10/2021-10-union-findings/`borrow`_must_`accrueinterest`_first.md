## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [`borrow` must `accrueInterest` first](https://github.com/code-423n4/2021-10-union-findings/issues/66) 

# Handle

cmichel


# Vulnerability details

The `UToken.borrow` function first checks the borrowed balance and the old credit limit _before_ accruing the actual interest on the market:

```solidity
// @audit this uses the old value
require(borrowBalanceView(msg.sender) + amount + fee <= maxBorrow, "UToken: amount large than borrow size max");

require(
    // @audit this calls uToken.calculateInterest(account) which returns old value
    uint256(_getCreditLimit(msg.sender)) >= amount + fee,
    "UToken: The loan amount plus fee is greater than credit limit"
);

// @audit accrual only happens here
require(accrueInterest(), "UToken: accrue interest failed");
```

Thus the borrowed balance of the user does not include the latest interest as it uses the old global `borrowIndex` but the new `borrowIndex` is only set in `accrueInterest`.

## Impact
In low-activity markets, it could be that the `borrowIndex` accruals (`accrueInterest` calls) happen infrequently and a long time is between them.
A borrower could borrow tokens, and borrow more tokens later at a different time without first having their latest debt accrued.
This will lead to borrowers being able to borrow more than `maxBorrow` and **more than their credit limit** as these checks are performed before updating accruing interest.

## Recommended Mitigation Steps
The `require(accrueInterest(), "UToken: accrue interest failed");` call should happen at the beginning of the function.


