## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Optimization on _redeemAndTransfer](https://github.com/code-423n4/2022-01-notional-findings/issues/213) 

# Handle

Tomio


# Vulnerability details

## Impact
There is unnecessary if else condition on _redeemAndTransfer(), and can be optimized by removing the inline if else condition on line https://github.com/code-423n4/2022-01-notional/blob/main/contracts/TreasuryAction.sol#L137-L140

## Proof of Concept
https://github.com/code-423n4/2022-01-notional/blob/main/contracts/TreasuryAction.sol#L137-L140

## Tools Used

## Recommended Mitigation Steps
From:
```
    if (underlying.tokenAddress == address(0)) {
            WETH9(WETH).deposit{value: address(this).balance}();
        }

        address underlyingAddress = underlying.tokenAddress == address(0)
            ? address(WETH)
            : underlying.tokenAddress;
        IERC20(underlyingAddress).safeTransfer(treasuryManagerContract, redeemedExternalUnderlying);
```

To:
```
    if (underlying.tokenAddress == address(0)) {
            WETH9(WETH).deposit{value: address(this).balance}();
            IERC20(WETH).safeTransfer(treasuryManagerContract, redeemedExternalUnderlying);
        }else{
            IERC20(underlying.tokenAddress).safeTransfer(treasuryManagerContract, redeemedExternalUnderlying);
        }
```

