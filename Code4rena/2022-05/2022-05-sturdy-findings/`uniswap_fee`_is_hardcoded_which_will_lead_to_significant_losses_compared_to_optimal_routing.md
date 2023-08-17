## Tags

- bug
- duplicate
- 2 (Med Risk)
- sponsor confirmed

# [`UNISWAP_FEE` is hardcoded which will lead to significant losses compared to optimal routing](https://github.com/code-423n4/2022-05-sturdy-findings/issues/48) 

# Lines of code

https://github.com/code-423n4/2022-05-sturdy/blob/78f51a7a74ebe8adfd055bdbaedfddc05632566f/smart-contracts/YieldManager.sol#L48
https://github.com/code-423n4/2022-05-sturdy/blob/78f51a7a74ebe8adfd055bdbaedfddc05632566f/smart-contracts/YieldManager.sol#L184


# Vulnerability details

## Impact
In [`YieldManager`](https://github.com/code-423n4/2022-05-sturdy/blob/78f51a7a74ebe8adfd055bdbaedfddc05632566f/smart-contracts/YieldManager.sol#L48), `UNISWAP_FEE` is hardcoded, which reduce significantly the possibilities and will lead to non optimal routes. In particular, all swaps using ETH path will use the wrong pool as it will use the ETH / USDC 1% one due to this [line](https://github.com/sturdyfi/code4rena-may-2022/blob/d53f4f5f0b7b33a66e0081294be6117f6d6e17b4/contracts/protocol/libraries/swap/UniswapAdapter.sol#L50). 


## Proof of Concept
For example for CRV / USDC, the optimal route is currently CRV -> ETH and ETH -> USDC, and the pool ETH / USDC with 1% fees is tiny compared to the ones with 0.3 or 0.1%. Therefore using the current implementation would create a significant loss of revenue.


## Recommended Mitigation Steps
Basic mitigation would be to hardcode in advance the best Uniswap paths in a mapping like it’s done for Curve pools, then pass this path already computed to the swapping library. This would allow for complex route and save gas costs as you would avoid computing them in `swapExactTokensForTokens`.

Then, speaking from experience, as `distributeYield` is `onlyAdmin`, you may want to add the possibility to do the swaps through an efficient aggregator like 1Inch or Paraswap, it will be way more optimal.

