## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas saving by struct reorganization](https://github.com/code-423n4/2021-12-sublime-findings/issues/14) 

# Handle

0x1f8b


# Vulnerability details

## Impact
Gas saving.

## Proof of Concept

It's possible to optimize the struct CreditLineConstants from CreditLine contract, the last 4 fields spend 3 storage slots, moving the boolean values between the address values, it will spend only two slots as follows:

```struct CreditLineConstants {
        address lender;
        address borrower;
        uint256 borrowLimit;
        uint256 idealCollateralRatio;
        uint256 borrowRate;
        address borrowAsset;
        bool autoLiquidation;
        address collateralAsset;
        bool requestByLender;
    }```.

## Tools Used
Manual review.

## Recommended Mitigation Steps
Reorder the structs fields

