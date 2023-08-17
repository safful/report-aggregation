## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed
- old-submission-method

# [TRSRY susceptible to loan / withdraw confusion](https://github.com/code-423n4/2022-08-olympus-findings/issues/75) 

# Lines of code

https://github.com/code-423n4/2022-08-olympus/blob/main/src/modules/TRSRY.sol#L64-L102


# Vulnerability details

## Impact
Treasury allocates approvals in the withdrawApproval mapping which is set via setApprovalFor(). In both withdrawReserves() and in getLoan(), _checkApproval() is used to verify user has enough approval and subtracts the withdraw / loan amount. Therefore, there is no differentiation in validation between loan approval and withdraw approval. Policies which will use getLoan() (currently none) can simply withdraw the tokens without bookkeeping it as a loan.

## Proof of Concept
1. Policy P has getLoan permission
2. setApprovalFor(policy, token, amount) was called to grant P permission to loan amount
3. P calls withdrawReserves(address, token, amount) and directly withdraws the funds without registering as loan

## Recommended Mitigation Steps
A separate mapping called loanApproval should be implemented, and setLoanApprovalFor() will set it, getLoan() will reduce loanApproval balance.

