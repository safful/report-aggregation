## Tags

- bug
- sponsor confirmed
- G (Gas Optimization)
- resolved

# [`Zapper.sol#tradeV3Single()` Remove unnecessary variable can make the code simpler and save gas](https://github.com/code-423n4/2021-10-ambire-findings/issues/28) 

# Handle

WatchPug


# Vulnerability details

At L149, `params.recipient` is read and put into a local variable `recipient`. However, `recipient` is only read once when `wrapOutputToLending` is true. Thus, the variable `recipient` is unnecessary.

https://github.com/code-423n4/2021-10-ambire/blob/bc01af4df3f70d1629c4e22a72c19e6a814db70d/contracts/wallet/Zapper.sol#L147-L159

```solidity=147
function tradeV3Single(ISwapRouter uniV3Router, ISwapRouter.ExactInputSingleParams calldata params, bool wrapOutputToLending) external returns (uint) {
    ISwapRouter.ExactInputSingleParams memory tradeParams = params;
    address recipient = params.recipient;
    if(wrapOutputToLending) {
        tradeParams.recipient = address(this);
    }

    uint amountOut = uniV3Router.exactInputSingle(tradeParams);
    if(wrapOutputToLending) {
        lendingPool.deposit(params.tokenOut, amountOut, recipient, aaveRefCode);
    }
    return amountOut;
}
```

### Recommendation

Change to:

```solidity=147
function tradeV3Single(ISwapRouter uniV3Router, ISwapRouter.ExactInputSingleParams calldata params, bool wrapOutputToLending) external returns (uint) {
    ISwapRouter.ExactInputSingleParams memory tradeParams = params;
    if(wrapOutputToLending) {
        tradeParams.recipient = address(this);
    }

    uint amountOut = uniV3Router.exactInputSingle(tradeParams);
    if(wrapOutputToLending) {
        lendingPool.deposit(params.tokenOut, amountOut, params.recipient, aaveRefCode);
    }
    return amountOut;
}
```

