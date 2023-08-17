## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [`setDebtInterestApr` should accrue debt first](https://github.com/code-423n4/2022-04-jpegd-findings/issues/78) 

# Lines of code

https://github.com/code-423n4/2022-04-jpegd/blob/e72861a9ccb707ced9015166fbded5c97c6991b6/contracts/vaults/NFTVault.sol#L212


# Vulnerability details

## Impact
The `setDebtInterestApr` changes the debt interest rate without first accruing the debt.
This means that the new debt interest rate is applied retroactively to the unaccrued period on next `accrue()` call.

It should never be applied retroactively to a previous time window as this is unfair & wrong.
Borrowers can incur more debt than they should.

## Recommended Mitigation Steps
Call `accrue()` first in `setDebtInterestApr` before setting the new `settings.debtInterestApr`.

