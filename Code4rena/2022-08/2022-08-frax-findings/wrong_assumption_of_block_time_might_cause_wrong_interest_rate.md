## Tags

- bug
- disagree with severity
- downgraded by judge
- QA (Quality Assurance)
- sponsor confirmed

# [Wrong assumption of block time might cause wrong interest rate](https://github.com/code-423n4/2022-08-frax-findings/issues/276) 

# Lines of code

https://github.com/code-423n4/2022-08-frax/blob/c4189a3a98b38c8c962c5ea72f1a322fbc2ae45f/src/contracts/FraxlendPairConstants.sol#L41
https://github.com/code-423n4/2022-08-frax/blob/c4189a3a98b38c8c962c5ea72f1a322fbc2ae45f/src/contracts/LinearInterestRate.sol#L34
https://github.com/code-423n4/2022-08-frax/blob/c4189a3a98b38c8c962c5ea72f1a322fbc2ae45f/src/contracts/VariableInterestRate.sol#L40
https://github.com/code-423n4/2022-08-frax/blob/c4189a3a98b38c8c962c5ea72f1a322fbc2ae45f/src/contracts/VariableInterestRate.sol#L41


# Vulnerability details

## Impact

All annual rate constants in the system are calculated with an assumption that block time is 15 second (actually it’s from 12 to 14 seconds as in [the documentation](https://ethereum.org/vi/developers/docs/blocks/#block-time). And these constants are used to calculate rate in rate calculator and also used to reset interest rate when there are no borrows.

But actually, [the merge is really near](https://ethereum.org/vi/upgrades/merge/) and after the merge blocks come exactly each 12 seconds which basically makes all these constants wrong. 

This resulted in wrong interest rate after reseting when there are no borrows and wrong rate returned by rate calculators.

## Proof of Concept

These annual rate is calculated by solving an equation for `r` with an assumption 365.24 days per year and 15s blocks. For example, this is for the 0.5% annual rate
```
1.005 = (1 + 15*r)^(365.24 * 24 * 3600 / 15)
```


But actually after the merge, blocks come in exactly each 12 seconds. Check out [this blog post](https://blog.ethereum.org/2021/11/29/how-the-merge-impacts-app-layer/) of Tim Beiko 

[Line 431-433](https://github.com/code-423n4/2022-08-frax/blob/c4189a3a98b38c8c962c5ea72f1a322fbc2ae45f/src/contracts/FraxlendPairCore.sol#L431-L433) reset interest rate when there are no borrows
```solidity
if (!paused()) {
    _currentRateInfo.ratePerSec = DEFAULT_INT;
}
```

These constants are used in `requireValidInitData()` and also `getNewRate()` function in rate calculators and wrong constants might make `getNewRate()` return wrong value. For example, [line 72-74](https://github.com/code-423n4/2022-08-frax/blob/c4189a3a98b38c8c962c5ea72f1a322fbc2ae45f/src/contracts/VariableInterestRate.sol#L72-L74) used `MIN_INT` as new interest rate
```solidity
if (_newRatePerSec < MIN_INT) {
    _newRatePerSec = MIN_INT;
}
```

## Tools Used

Manual Review

## Recommended Mitigation Steps

Consider to update these constants with an assumption that block time is 12 seconds.
