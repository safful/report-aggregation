## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Stop ramp target price would create huge arbitrage space.](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/208) 

# Handle

jonah1005


# Vulnerability details

# Stop ramp target price would create huge arbitrage space.
## Impact
`stopRampTargetPrice` would set the `tokenPrecisionMultipliers` to `originalPrecisionMultipliers[0].mul(currentTargetPrice).div(WEI_UNIT);`
Once the `tokenPrecisionMultipliers` is changed, the price in the AMM pool would change. Arbitrager can sandwich `stopRampTargetPrice` to gain profit.

Assume the decision is made in the DAO, an attacker can set up the bot once the proposal to `stopRampTargetPrice` has passed. I consider this is a medium-risk issue.

## Proof of Concept
The `precisionMultiplier` is set here:
[Swap.sol#L661-L666](https://github.com/code-423n4/2021-11-bootfinance/blob/main/customswap/contracts/Swap.sol#L661-L666)

We can set up a mockSwap with extra `setPrecisionMultiplier` to check the issue.
```solidity
    function setPrecisionMultiplier(uint256 multipliers) external {
        swapStorage.tokenPrecisionMultipliers[0] = multipliers; 
    }
```

```python
print(swap.functions.getVirtualPrice().call())
swap.functions.setPrecisionMultiplier(2).transact()
print(swap.functions.getVirtualPrice().call())

# output log:
#     1000000000000000000
#     1499889859738721606
```
## Tools Used
None
## Recommended Mitigation Steps
Dealing with the target price with multiplier precision seems clever as we can reuse most of the existing code. However, the precision multiplier should be an immutable parameter. Changing it after the pool is setup would create multiple issues. This function could be implemented in a safer way IMHO.

A quick fix I would come up with is to ramp the `tokenPrecisionMultipliers` as the `aPrecise` is ramped. As the `tokenPrecision` is slowly increased/decreased, the arbitrage space would be slower and the profit would (probably) distribute evenly to lpers.

Please refer to `_getAPreceise`'s implementation
[SwapUtils.sol#L227-L250](https://github.com/code-423n4/2021-11-bootfinance/blob/main/customswap/contracts/SwapUtils.sol#L227-L250)

