## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas Optimization: Struct layout](https://github.com/code-423n4/2021-12-sublime-findings/issues/129) 

# Handle

gzeon


# Vulnerability details

Rewrite the PoolConstants struct as follow can save some gas
```
    struct PoolConstants {
        uint256 borrowAmountRequested;
        uint256 loanStartTime;
        uint256 loanWithdrawalDeadline;
        uint256 idealCollateralRatio;
        uint256 borrowRate;
        uint256 noOfRepaymentIntervals;
        uint256 repaymentInterval;
        address borrower;
        address borrowAsset;
        address collateralAsset;
        address poolSavingsStrategy; // invest contract
        address lenderVerifier;
    }
    
```
https://github.com/code-423n4/2021-12-sublime/blob/9df1b7c4247f8631647c7627a8da9bdc16db8b11/contracts/Pool/Pool.sol#L46

