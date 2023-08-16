## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Change in interest rate can disable repay of loan](https://github.com/code-423n4/2021-10-union-findings/issues/21) 

# Handle

pmerkleplant


# Vulnerability details

## Impact
The ability of a borrower to repay a loan is disabled if the interest rate is 
set too high by the `InterestRateModel`.

However, there is neither a check when setting the interest rate nor an 
indication in the `IInterestRateModel`'s specs of this behavior.

But this issue could also be used in an adversarial fashion by the 
`FixedInterestRateModel`-owner if he/she would disable the repay functionality 
for some time and enables it at a later point again with the demand of a 
higher interest to be paid by the borrower.

## Proof of Concept
If an account wants to repay a loan, the function 
`UToken::_repayBorrowFresh()` is used. This function calls 
`UToken::accrueInterest()` ([line](https://github.com/code-423n4/2021-10-union/blob/main/contracts/market/UToken.sol#L465) 465) 
which fetches the current borrow rate of the interest rate model 
([line](https://github.com/code-423n4/2021-10-union/blob/main/contracts/market/UToken.sol#L546) 546 
and [line](https://github.com/code-423n4/2021-10-union/blob/main/contracts/market/UToken.sol#L330) 330).

The function `UToken::borrowRatePerBlock()` requires an not "absurdly high" 
rate, or fails otherwise ([line](https://github.com/code-423n4/2021-10-union/blob/main/contracts/market/UToken.sol#L331) 331).

However, there is no check or indicator in `FixedInterestRateModel.sol` to 
prevent the owner to set such a high rate that effectively disables repay
of borrowed funds ([line](https://github.com/code-423n4/2021-10-union/blob/main/contracts/market/FixedInterestRateModel.sol#L36) 36).

## Recommended Mitigation Steps
Disallow setting the interest rate too high with a check in 
`FixedInterestRateModel::setInterestRate()`.

