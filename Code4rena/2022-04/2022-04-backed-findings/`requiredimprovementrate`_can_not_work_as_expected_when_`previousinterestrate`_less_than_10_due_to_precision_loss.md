## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [`requiredImprovementRate` can not work as expected when `previousInterestRate` less than 10 due to precision loss](https://github.com/code-423n4/2022-04-backed-findings/issues/80) 

# Lines of code

https://github.com/code-423n4/2022-04-backed/blob/e8015d7c4b295af131f017e646ba1b99c8f608f0/contracts/NFTLoanFacilitator.sol#L167-L179


# Vulnerability details


https://github.com/code-423n4/2022-04-backed/blob/e8015d7c4b295af131f017e646ba1b99c8f608f0/contracts/NFTLoanFacilitator.sol#L167-L179

```solidity
{
    uint256 previousInterestRate = loan.perAnumInterestRate;
    uint256 previousDurationSeconds = loan.durationSeconds;

    require(interestRate <= previousInterestRate, 'NFTLoanFacilitator: rate too high');
    require(durationSeconds >= previousDurationSeconds, 'NFTLoanFacilitator: duration too low');

    require((previousLoanAmount * requiredImprovementRate / SCALAR) <= amountIncrease
    || previousDurationSeconds + (previousDurationSeconds * requiredImprovementRate / SCALAR) <= durationSeconds 
    || (previousInterestRate != 0 // do not allow rate improvement if rate already 0
        && previousInterestRate - (previousInterestRate * requiredImprovementRate / SCALAR) >= interestRate), 
    "NFTLoanFacilitator: proposed terms must be better than existing terms");
}
```

The `requiredImprovementRate` represents the percentage of improvement required of at least one of the terms when buying out from a previous lender.

However, when `previousInterestRate` is less than `10` and `requiredImprovementRate` is `100`, due to precision loss, the new `interestRate` is allowed to be the same as the previous one.

Making such an expected constraint absent.

### PoC

1. Alice `createLoan()` with `maxPerAnumInterest` = 10, received `loanId` = 1
2. Bob `lend()` with `interestRate` = 9  for `loanId` = 1
3. Charlie `lend()` with `interestRate` = 9 (and all the same other terms with Bob) and buys out `loanId` = 1

Charlie is expected to provide at least 10% better terms, but actually bought out Bob with the same terms.

### Recommendation

Consider using:
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.5.0/contracts/utils/math/Math.sol#L39-L42

And change the check to:

```solidity
(previousInterestRate != 0 // do not allow rate improvement if rate already 0
        && previousInterestRate - Math.ceilDiv(previousInterestRate * requiredImprovementRate, SCALAR) >= interestRate)
```


