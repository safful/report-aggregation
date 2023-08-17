## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Mistake while checking LTV to lender accepted LTV](https://github.com/code-423n4/2022-04-abranft-findings/issues/55) 

# Lines of code

https://github.com/code-423n4/2022-04-abranft/blob/main/contracts/NFTPairWithOracle.sol#L316


# Vulnerability details

## Impact
It comments in the _lend() function that lender accepted conditions must be at least as good as the borrower is asking for.
The line which checks the accepted LTV (lender's LTV) against borrower asking LTV is:
                params.ltvBPS >= accepted.ltvBPS,
This means lender should be offering a lower LTV, which must be the opposite way around. 
I think this may have the potential to strand the lender, if he enters a lower LTV.
For example borrower asking LTV is 86%. However, lender enters his accepted LTV as 80%.
lend() will execute with 86% LTV and punish the lender, whereas it should revert and acknowledge the lender that his bid is not good enough.

## Proof of Concept
https://github.com/code-423n4/2022-04-abranft/blob/main/contracts/NFTPairWithOracle.sol#L316

## Tools Used
Manual analysis

## Recommended Mitigation Steps
The condition should be changed as:
                params.ltvBPS <= accepted.ltvBPS,



