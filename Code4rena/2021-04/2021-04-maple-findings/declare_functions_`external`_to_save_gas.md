## Tags

- bug
- sponsor confirmed
- resolved
- G (Gas Optimization)

# [Declare functions `external` to save gas](https://github.com/code-423n4/2021-04-maple-findings/issues/11) 

# Handle

JMukesh


# Vulnerability details

// All these function described should be declared  external, as functions that are never called by the contract should be declared external to save gas. 
              
In fundingLockerFactory.sol --> newLocker(){}
In LatefeeCalc.sol          --> getlateFee(){]
In Loan.sol                 --> MakeFullPayment(){}
In library/Loanlib.sol      --> getNextPayment(){}
In library/Util.sol         --> calcMinAmount(){}
In token/BasicFDT.sol       --> withdrawnFundsOf(){}
In MapleTreasury.sol        --> reclaimERC20(){}
                               distributeToHolder(){}
                               convertERC20(){}

In Pool.sol                 --> claimablefunds(){}
                            --> BPTval(){}
In Poollib,sol              --> validateDeactivation(){}
                               isWithdrawAllowed(){}
                               getInitialStakeRequirements(){}
                               ecognizedLossesOf(){}
In Premiumcal.sol            --> getPremium(){}
In Repayment.sol            -->  getNextPayment(){}




