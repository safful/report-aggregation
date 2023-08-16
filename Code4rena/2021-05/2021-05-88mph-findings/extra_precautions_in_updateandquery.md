## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- resolved

# [Extra precautions in updateAndQuery](https://github.com/code-423n4/2021-05-88mph-findings/issues/10) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function updateAndQuery of EMAOracle.sol subtracts the incomeIndex with the previous incomeIndex.
These incomeIndex values are retrieved via the moneyMarket contract from an external contract.

If by accident the previous incomeIndex is larger than the current incomeIndex then the subtraction would be negative and the code halts (reverts), without an error message.
Also the updateAndQuery function would not be able to execute (until the current incomeIndex is larger than the previous incomeIndex).

This situation could occur when an error occurs in one of the current or future money markets.

## Proof of Concept
EMAOracle.sol:
  function updateAndQuery() {
        ...
        uint256 _lastIncomeIndex = lastIncomeIndex;
        ...
        uint256 newIncomeIndex = moneyMarket.incomeIndex();
        uint256 incomingValue =
            (newIncomeIndex - _lastIncomeIndex).decdiv(_lastIncomeIndex) /
                timeElapsed;

## Tools Used
Editor

## Recommended Mitigation Steps
Give an error message when the previous incomeIndex is larger than the current incomeIndex.
And/or create a way to recover from this erroneous situation.

