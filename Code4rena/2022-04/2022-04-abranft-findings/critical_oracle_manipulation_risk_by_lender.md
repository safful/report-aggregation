## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Critical Oracle Manipulation Risk by Lender](https://github.com/code-423n4/2022-04-abranft-findings/issues/37) 

# Lines of code

https://github.com/code-423n4/2022-04-abranft/blob/5cd4edc3298c05748e952f8a8c93e42f930a78c2/contracts/NFTPairWithOracle.sol#L286-L288
https://github.com/code-423n4/2022-04-abranft/blob/5cd4edc3298c05748e952f8a8c93e42f930a78c2/contracts/NFTPairWithOracle.sol#L200-L211


# Vulnerability details

## Impact

The intended use of the Oracle is to protect the lender from a drop in the borrower's collateral value. If the collateral value goes up significantly and higher than borrowed amount + interest, the lender should not be able to seize the collateral at the expense of the borrower. However, in the `NFTPairWithOracle` contract, the lender could change the Oracle once a loan is outstanding, and therefore seize the collateral at the expense of the borrower, if the actual value of the collateral has increased significantly. This is a critical risk because borrowers asset could be lost to malicious lenders.


## Proof of Concept

In `NFTPairWithOracle`, the `params` are set by the `borrower` when they call `requestLoan()`, including the Oracle used. Once a lender agrees with the parameters and calls the `lend()` function, the `loan.status` changes to `LOAN_OUTSTANDING`. 

Then, the lender can call the `updateLoanParams()` function and pass in its own `params` including the Oracle used. The `require` statement from line 205 to 211 does not check if `params.oracle` and `cur.oracle` are the same. A malicious lender could pass in his own `oracle` after the loan becomes outstanding, and the change would be reflected in line 221. 

https://github.com/code-423n4/2022-04-abranft/blob/5cd4edc3298c05748e952f8a8c93e42f930a78c2/contracts/NFTPairWithOracle.sol#L200-L211

In a situation where the actual value of the collateral has gone up by a lot, exceeding the amount the lender is owed (principal + interest), the lender would have an incentive to seize the collateral. If the Oracle is not tampered with, lender should not be able to do this, because line 288 should fail. But a lender could freely change Oracle once the loan is outstanding, then a tampered Oracle could produce a very low `rate` in line 287 such that line 288 would pass, allowing the lender to seize the collateral, hurting the borrower. 

https://github.com/code-423n4/2022-04-abranft/blob/5cd4edc3298c05748e952f8a8c93e42f930a78c2/contracts/NFTPairWithOracle.sol#L286-L288



## Tools Used

Manual review

## Recommended Mitigation Steps

Once a loan is agreed to, the oracle used should not change. I'd recommend adding a check in the `require` statement in line 205 - 211 that `params.oracle == cur.oracle`

