## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Get virtual price is not monotonically increasing ](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/185) 

# Handle

jonah1005


# Vulnerability details

## Impact
There's a feature of `virtualPrice` that is monotonically increasing regardless of the market. This function is heavily used in multiple protocols. e.g.(curve metapool, mim, ...) This is not held in the current implementation of customSwap since `customPrecisionMultipliers` can be changed by changing the target price.

There're two issues here:
The meaning of `virtualPrice` would be vague.
This may damage the lp providers as the protocol that adopts it may be hacked.

I consider this is a medium-risk issue.

## Proof of Concept
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
#   1000000000000000000
#   1499889859738721606
```
## Tools Used
None
## Recommended Mitigation Steps
Dealing with the target price with multiplier precision seems clever as we can reuse most of the existing code. However, the precision multiplier should be an immutable parameter. Changing it after the pool is set up would create multiple issues. This function could be implemented in a safer way IMHO.

The quick fix would be to remove the `getVirtualPrice` function. I can't come up with a safe way if other protocol wants to use this function.


