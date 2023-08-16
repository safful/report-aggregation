## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Wrong value for `SwappedTokens` event parameter](https://github.com/code-423n4/2021-10-tally-findings/issues/28) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-10-tally/blob/c585c214edb58486e0564cb53d87e4831959c08b/contracts/swap/Swap.sol#L174-L180

```solidity
emit SwappedTokens(
    zrxSellTokenAddress,
    zrxBuyTokenAddress,
    amountToSell,
    boughtETHAmount,
    boughtETHAmount.sub(toTransfer)
);
```

`amountToSell` will be 0 according to the comment: `If selling unwrapped ETH via msg.value, this should be 0.`, therefore, `msg.value` should be used instead.

### Recommendation

Change to:

```solidity
emit SwappedTokens(
    zrxSellTokenAddress,
    zrxBuyTokenAddress,
    msg.value,
    boughtETHAmount,
    boughtETHAmount.sub(toTransfer)
);
```

