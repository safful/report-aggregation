## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- resolved

# [Unused definition of enum](https://github.com/code-423n4/2021-04-maple-findings/issues/17) 

# Handle

gpersoon


# Vulnerability details

## Impact

LoanLib.sol has a definition of enum State and Loan.sol has the same definition.
The LoanLib.sol does not seem to be used
This means dead code and could be confusing.

## Proof of Concept

Loan.sol:       enum State { Ready, Active, Matured, Expired, Liquidated }
LoanLib.sol:    enum State { Ready, Active, Matured, Expired, Liquidated }

## Tools Used

grep "enum" *.sol -S

## Recommended Mitigation Steps

Remove the unused definition from LoanLib.sol
(or make sure there is just one definition for the enum and include that elsewhere)

