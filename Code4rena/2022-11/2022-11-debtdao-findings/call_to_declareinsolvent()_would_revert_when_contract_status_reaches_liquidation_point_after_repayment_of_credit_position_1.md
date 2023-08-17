## Tags

- bug
- 3 (High Risk)
- primary issue
- satisfactory
- sponsor confirmed
- upgraded by judge
- selected for report
- edited-by-warden
- H-01

# [Call to declareInsolvent() would revert when contract status reaches liquidation point after repayment of credit position 1](https://github.com/code-423n4/2022-11-debtdao-findings/issues/69) 

# Lines of code

https://github.com/debtdao/Line-of-Credit/blob/audit/code4rena-2022-11-03/contracts/modules/credit/LineOfCredit.sol#L143
https://github.com/debtdao/Line-of-Credit/blob/audit/code4rena-2022-11-03/contracts/modules/credit/LineOfCredit.sol#L83-L86


# Vulnerability details

## Impact
The modifier `whileBorrowing()` is used along in the call to LineOfCredit.declareInsolvent(). However this check reverts when count == 0 or `credits[ids[0]].principal == 0` . Within the contract, any lender can add credit which adds an entry in credits array, credits[ids]. 

Assume, when borrower chooses lender positions including credits[ids[0]] to draw on, and repays back the loan fully for credits[ids[1]], then the call to declareInsolvent() by the arbiter would revert since it does not pass the `whileBorrowing()` modifier check due to the ids array index shift in the call to  stepQ(), which would shift ids[1] to ids[0], thereby making the condition for `credits[ids[0]].principal == 0` be true causing the revert.



## Proof of Concept
1. LineOfCredit contract is set up and 5 lenders have deposited into the contract.
2. Alice, the borrower borrows credit from these 5 credit positions including by calling LineOfCredit.borrow() for the position ids.
3. Later Alice pays back the loan for  credit position id 1 just before the contract gets liquidated
4. At the point where ids.stepQ() is called in _repay(), position 1 is moved to ids[0]
4. When contract status is LIQUIDATABLE, no loan drawn on credit position 0 and arbiter calls declareInsolvent() , the call would revert since `credits[ids[0]].principal == 0`

## Tools Used
Manual review

## Recommended Mitigation Steps
The modifier whileBorrowing() would need to be reviewed and amended.