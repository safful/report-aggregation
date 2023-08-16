## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [`SwapUtils.sol` Inconsistent parameter value of `lpTokenSupply` among `Liquidity` related events](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/237) 

# Handle

WatchPug


# Vulnerability details

There are 4 events with the parameter `lpTokenSupply` in `SwapUtils.sol`, but the value of `lpTokenSupply` is not consistent.

For the event `RemoveLiquidityOne`, `lpTokenSupply` is post burn:

https://github.com/code-423n4/2021-11-bootfinance/blob/f102ee73eb320532c5a7c1e833f225c479577e39/customswap/contracts/SwapUtils.sol#L1349-L1349

```solidity=1349
emit RemoveLiquidity(msg.sender, amounts, self.lpToken.totalSupply());
```

For the event `RemoveLiquidityOne`, `lpTokenSupply` is pre burn:

https://github.com/code-423n4/2021-11-bootfinance/blob/f102ee73eb320532c5a7c1e833f225c479577e39/customswap/contracts/SwapUtils.sol#L1368-L1368

For the event `removeLiquidityImbalance`, `lpTokenSupply` is post burn:

https://github.com/code-423n4/2021-11-bootfinance/blob/f102ee73eb320532c5a7c1e833f225c479577e39/customswap/contracts/SwapUtils.sol#L1475-L1481

For the event `AddLiquidity`, `lpTokenSupply` is post mint:

https://github.com/code-423n4/2021-11-bootfinance/blob/f102ee73eb320532c5a7c1e833f225c479577e39/customswap/contracts/SwapUtils.sol#L1261-L1267

### Recommendation

Given that 3 out of the 4 events are using updated `totalSupply` as `lpTokenSupply`, consider changing `RemoveLiquidityOne` to post burn `totalSupply`.

