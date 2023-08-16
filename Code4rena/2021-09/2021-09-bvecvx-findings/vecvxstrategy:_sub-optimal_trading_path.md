## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [veCVXStrategy: Sub-optimal trading path](https://github.com/code-423n4/2021-09-bvecvx-findings/issues/38) 

# Handle

hickuphh3


# Vulnerability details

### Impact

`_swapcvxCRVToWant()` swaps `cvxCRV -> ETH -> CVX` via sushiswap.

Looking at sushiswap analytics, this may also not be the most optimal trading path. The cvxCRV-CRV pool seems to have substantially better liquidity than the cvxCRV-ETH pool as r[eported here](https://www.notion.so/6a2dc64a1969e19c23e4f579f9810aa7) (Note that cvxCRV-CRV's liquidity is overstated, [clicking into the pool](https://www.notion.so/a2a8a54062e021873bcaee006cdf4007) gives a more reasonable amount). It is therefore better to do `cvxCRV -> CRV -> ETH -> CVX`, though this comes at the cost of higher gas usage.

### Recommended Mitigation Steps

Switch the trading path to `cvxCRV -> CRV -> ETH -> CVX`, as it means more CVX tokens received, translating to higher APY, while the higher gas cost is borne by the caller.

Additionally, given how liquidity can shift between pools over time, the most optimal trade path may change accordingly. Hence, it may be beneficial to make the pool path configurable.

