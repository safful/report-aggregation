## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Funding.deposit() doesn't work if there is no discount set](https://github.com/code-423n4/2022-04-badger-citadel-findings/issues/149) 

# Lines of code

https://github.com/code-423n4/2022-04-badger-citadel/blob/main/src/Funding.sol#L177
https://github.com/code-423n4/2022-04-badger-citadel/blob/main/src/Funding.sol#L202
https://github.com/code-423n4/2022-04-badger-citadel/blob/main/src/Funding.sol#L184
https://github.com/code-423n4/2022-04-badger-citadel/blob/main/src/StakedCitadel.sol#L769


# Vulnerability details

## Impact
The Funding contract's `deposit()` function uses the `getAmountOut()` function to determine how many citadel tokens the user should receive for their deposit. But, if no discount is set, the function always returns 0. Now the `deposit()` function tries to deposit 0 tokens for the user through the StakedCitadel contract. But, that function requires the number of tokens to be `!= 0`. The transaction reverts.

This means, that no deposits are possible. Unless there is a discount.

## Proof of Concept
`Funding.deposit()` calls `getAmountOut()`: https://github.com/code-423n4/2022-04-badger-citadel/blob/main/src/Funding.sol#L177

Here's the [`getAmountOut()` function](https://github.com/code-423n4/2022-04-badger-citadel/blob/main/src/Funding.sol#L202):
```sol
    function getAmountOut(uint256 _assetAmountIn)
        public
        view
        returns (uint256 citadelAmount_)
    {
        uint256 citadelAmountWithoutDiscount = _assetAmountIn * citadelPriceInAsset;

        if (funding.discount > 0) {
            citadelAmount_ =
                (citadelAmountWithoutDiscount * MAX_BPS) /
                (MAX_BPS - funding.discount);
        }

        // unless the above if block is executed, `citadelAmount_` is 0 when this line is executed.
        // 0 = 0 / x
        citadelAmount_ = citadelAmount_ / assetDecimalsNormalizationValue;
    }
```

Call to `StakedCitadel.depositFor()`: https://github.com/code-423n4/2022-04-badger-citadel/blob/main/src/Funding.sol#L184

require statement that makes the whole transaction revert: https://github.com/code-423n4/2022-04-badger-citadel/blob/main/src/StakedCitadel.sol#L769

## Tools Used
none

## Recommended Mitigation Steps
Change the `getAmountOut()` function to:

```sol
    function getAmountOut(uint256 _assetAmountIn)
        public
        view
        returns (uint256 citadelAmount_)
    {

        uint256 citadelAmount_ = _assetAmountIn * citadelPriceInAsset;

        if (funding.discount > 0) {
            citadelAmount_ =
                (citadelAmount_ * MAX_BPS) /
                (MAX_BPS - funding.discount);
        }

        citadelAmount_ = citadelAmount_ / assetDecimalsNormalizationValue;
    }
```

