## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Lender is able to seize the collateral by changing the loan parameters](https://github.com/code-423n4/2022-04-abranft-findings/issues/51) 

# Lines of code

https://github.com/code-423n4/2022-04-abranft/blob/main/contracts/NFTPairWithOracle.sol#L198-L223
https://github.com/code-423n4/2022-04-abranft/blob/main/contracts/NFTPairWithOracle.sol#L200-L212
https://github.com/code-423n4/2022-04-abranft/blob/main/contracts/NFTPairWithOracle.sol#L288


# Vulnerability details

## Impact
The lender should only be able to seize the collateral if:
- the borrower didn't repay in time
- the collateral loses too much of its value

But, the lender is able to seize the collateral at any time by modifying the loan parameters.

## Proof of Concept
The [`updateLoanParams()`](https://github.com/code-423n4/2022-04-abranft/blob/main/contracts/NFTPairWithOracle.sol#L198-L223) allows the lender to modify the parameters of an active loan in favor of the borrower. But, by setting the `ltvBPS` value to `0` they are able to seize the collateral.

If `ltvBPS` is `0` the following require statement in `removeCollateral()` will always be true:

https://github.com/code-423n4/2022-04-abranft/blob/main/contracts/NFTPairWithOracle.sol#L288

`rate * 0 / BPS < amount` is always `true`.

That allows the lender to seize the collateral although its value didn't decrease nor did the time to repay the loan come.

So the required steps are:
1. lend the funds to the borrower
2. call `updateLoanParams()` to set the `ltvBPS` value to `0`
3. call `removeCollateral()` to steal the collateral from the contract

## Tools Used
none

## Recommended Mitigation Steps
Don't allow `updateLoanParams()` to change the `ltvBPS` value.

