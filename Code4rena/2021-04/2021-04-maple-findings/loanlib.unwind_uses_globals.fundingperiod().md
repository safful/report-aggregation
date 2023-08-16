## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- resolved

# [LoanLib.unwind uses globals.fundingPeriod()](https://github.com/code-423n4/2021-04-maple-findings/issues/100) 

# Handle

paulius.eth


# Vulnerability details

## Vulnerability details

Every loan has its own fundingPeriod which is set once in the constructor:
fundingPeriod          = globals.fundingPeriod();
fundingPeriod in globals can change. It does not effect already deployed Loans.
However, in Loan contract function unwind() calls LoanLib.unwind which checks against globals.fundingPeriod():
        IGlobals globals = _globals(superFactory);
        // Only callable if time has passed drawdown grace period, set in MapleGlobals
        require(block.timestamp > createdAt.add(globals.fundingPeriod()), "Loan:FUNDING_PERIOD_NOT_FINISHED");
at this time, globals.fundingPeriod() could be different than this specific Loan's fundingPeriod.

## Recommended Mitigation Steps

Check expiration against local fundingPeriod.


