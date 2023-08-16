## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [No validation of protocol fee fraction](https://github.com/code-423n4/2021-12-sublime-findings/issues/84) 

# Handle

sirhashalot


# Vulnerability details

## Impact

The `updateProtocolFeeFraction` function in CreditLine.sol does not validate the value submitted. Fee fractions of 0%, 100%, or 200% are equally valid. A maximum fee value check is recommended and a similar check is used in `_updateLiquidatorRewardFraction` in CreditLine.sol to set a maximum liquidator fraction. However, if the assumption is that the owner is trusted and does not make mistakes, this may not be considered a problem.
 
## Proof of Concept

The `updateProtocolFeeFraction` function calls `_updateProtocolFeeFraction` in CreditLine.sol:
https://github.com/code-423n4/2021-12-sublime/blob/9df1b7c4247f8631647c7627a8da9bdc16db8b11/contracts/CreditLine/CreditLine.sol#L335-L338 

## Tools Used

Manual analysis

## Recommended Mitigation Steps

Apply a maximum fee hard cap with a require statement to make sure the fee does not exceed a certain limit, whether by admin error or theoretical malicious overtake of the contract

