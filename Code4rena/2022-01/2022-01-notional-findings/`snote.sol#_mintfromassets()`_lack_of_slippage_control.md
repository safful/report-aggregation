## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [`sNOTE.sol#_mintFromAssets()` Lack of slippage control](https://github.com/code-423n4/2022-01-notional-findings/issues/181) 

# Handle

WatchPug


# Vulnerability details

ttps://github.com/code-423n4/2022-01-notional/blob/d171cad9e86e0d02e0909eb66d4c24ab6ea6b982/contracts/sNOTE.sol#L195-L209

```solidity
BALANCER_VAULT.joinPool{value: msgValue}(
    NOTE_ETH_POOL_ID,
    address(this),
    address(this), // sNOTE will receive the BPT
    IVault.JoinPoolRequest(
        assets,
        maxAmountsIn,
        abi.encode(
            IVault.JoinKind.EXACT_TOKENS_IN_FOR_BPT_OUT,
            maxAmountsIn,
            0 // Accept however much BPT the pool will give us
        ),
        false // Don't use internal balances
    )
);
```

The current implementation of `mintFromNOTE()` and `mintFromETH()` and `mintFromWETH()` (all are using `_mintFromAssets()` with `minimumBPT` hardcoded to `0`) provides no parameter for slippage control, making it vulnerable to front-run attacks.

### Recommendation

Consider adding a `minAmountOut` parameter for these functions.

