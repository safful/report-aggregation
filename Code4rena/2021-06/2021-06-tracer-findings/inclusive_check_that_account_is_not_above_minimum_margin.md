## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [inclusive check that account is not above minimum margin](https://github.com/code-423n4/2021-06-tracer-findings/issues/109) 

# Handle

pauliax


# Vulnerability details

## Impact
Here the check currentMargin < Balances.minimumMargin should be inclusive <= to indicate the account is not above minimum margin:
    require(
        currentMargin <= 0 ||
            uint256(currentMargin) < Balances.minimumMargin(pos, price, gasCost, tracer.trueMaxLeverage()),
        "LIQ: Account above margin"
    );

## Recommended Mitigation Steps
uint256(currentMargin) <= Balances.minimumMargin
...

