## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas: minReceived check can be simplified](https://github.com/code-423n4/2021-10-tally-findings/issues/41) 

# Handle

cmichel


# Vulnerability details

The `minimumAmountReceived` check in `Swap.swapByQuote` is implemented like this:

```solidity
require(
    (
        !signifiesETHOrZero(zrxBuyTokenAddress) &&
        boughtERC20Amount >= minimumAmountReceived
    ) ||
    (
        signifiesETHOrZero(zrxBuyTokenAddress) &&
        boughtETHAmount >= minimumAmountReceived
    ),
    "Swap::swapByQuote: Minimum swap proceeds requirement not met"
);
```

It can be simplified to this which performs less calls to `signifiesETHOrZero` and less logical operators:

```solidity
require( (signifiesETHOrZero(zrxBuyTokenAddress) ? boughtETHAmount : boughtERC20Amount) >= minimumAmountReceived, "...");
```


