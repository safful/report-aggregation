## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- sponsor confirmed
- selected for report
- M-02

# [Mutual consent cannot be revoked and stays valid forever](https://github.com/code-423n4/2022-11-debtdao-findings/issues/33) 

# Lines of code

https://github.com/debtdao/Line-of-Credit/blob/e8aa08b44f6132a5ed901f8daa231700c5afeb3a/contracts/utils/MutualConsent.sol#L11-L68
https://github.com/debtdao/Line-of-Credit/blob/e8aa08b44f6132a5ed901f8daa231700c5afeb3a/contracts/modules/credit/LineOfCredit.sol#L247-L262


# Vulnerability details

## Impact
Contracts that inherit from the `MutualConsent` contract, have access to a `mutualConsent` modifier.  
Functions that use this modifier need consent from two parties to be called successfully.  

Once one party has given consent for a function call, it cannot revoke consent.  
This means that the other party can call this function at any time now.  

This opens the door for several exploitation paths.  
Most notably though the functions `LineOfCredit.setRates()`, `LineOfCredit.addCredit()` and `LineOfCredit.increaseCredit()` can cause problems.  

One party can use Social Engineering to make the other party consent to multiple function calls and exploit the multiple consents.  

## Proof of Concept
1. A borrower and lender want to change the rates for a credit.  
   The borrower wants to create the possibility for himself to change the rates in the future without the lender's consent.  
2. The borrower and lender agree to set `dRate` and `fRate` to 5%.
3. The lender calls the `LineOfCredit.setRates()` function to give his consent.
4. The borrower might now say to the lender "Let's put the rate to 5.1% instead, I will give an extra 0.1%"
5. The borrower and lender now both call the `LineOfCredit.setRates()` function to set the rates to 5.1%.
6. The borrower can now set the rates to 5% at any time. E.g. they might increase the rates further in the future (the borrower playing by the rules)  
   and at some point the borrower can decide to set the rates to 5%.

Links:  
`MutualConsent` contract: [https://github.com/debtdao/Line-of-Credit/blob/audit/code4rena-2022-11-03/contracts/utils/MutualConsent.sol](https://github.com/debtdao/Line-of-Credit/blob/audit/code4rena-2022-11-03/contracts/utils/MutualConsent.sol)  

`LineOfCredit.setRates()` function: [https://github.com/debtdao/Line-of-Credit/blob/e8aa08b44f6132a5ed901f8daa231700c5afeb3a/contracts/modules/credit/LineOfCredit.sol#L247-L262](https://github.com/debtdao/Line-of-Credit/blob/e8aa08b44f6132a5ed901f8daa231700c5afeb3a/contracts/modules/credit/LineOfCredit.sol#L247-L262)


## Tools Used
VSCode

## Recommended Mitigation Steps
There are several options to fix this issue:
1. Add a function to the `MutualConsent` contract to revoke consent for a function call.
2. Make consent valid only for a certain amount of time.
3. Invalidate existing consents for a function when function is called with different arguments.

Option 3 requires a lot of additional bookkeeping but is probably the cleanest solution.