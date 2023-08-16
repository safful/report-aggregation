## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Ideal balance is not calculated correctly when providing imbalanced liquidity](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/150) 

# Handle

jonah1005


# Vulnerability details

## Impact
When a user provides imbalanced liquidity, the fee is calculated according to the ideal balance. In saddle finance, the optimal balance should be the same ratio as in the Pool.

Take, for example, if there's 10000 USD and 10000 DAI in the saddle's USD/DAI pool, the user should get the optimal lp if he provides lp with ratio = 1.

However, if the customSwap pool is created with a target price = 2. The user would get 2 times more lp if he deposits DAI.
[SwapUtils.sol#L1227-L1245](https://github.com/code-423n4/2021-11-bootfinance/blob/main/customswap/contracts/SwapUtils.sol#L1227-L1245)
The current implementation does not calculates ideal balance correctly.

If the target price is set to be 10, the ideal balance deviates by 10.
The fee deviates a lot. I consider this is a high-risk issues.

## Proof of Concept
We can observe the issue if we initiates two pools DAI/LINK pool and set the target price to be 4.

For the first pool, we deposit more dai.
```python
    swap = deploy_contract('Swap' 
        [dai.address, link.address], [18, 18], 'lp', 'lp', 1, 85, 10**7, 0, 0, 4* 10**18)
    link.functions.approve(swap.address, deposit_amount).transact()
    dai.functions.approve(swap.address, deposit_amount).transact()
    previous_lp = lptoken.functions.balanceOf(user).call()
    swap.functions.addLiquidity([deposit_amount, deposit_amount // 10], 10, 10**18).transact()
    post_lp = lptoken.functions.balanceOf(user).call()
    print('get lp', post_lp - previous_lp)
```

For the second pool, one we deposit more dai.
```python
    swap = deploy_contract('Swap' 
        [dai.address, link.address], [18, 18], 'lp', 'lp', 1, 85, 10**7, 0, 0, 4* 10**18)
    link.functions.approve(swap.address, deposit_amount).transact()
    dai.functions.approve(swap.address, deposit_amount).transact()
    previous_lp = lptoken.functions.balanceOf(user).call()
    swap.functions.addLiquidity([deposit_amount, deposit_amount // 10], 10, 10**18).transact()
    post_lp = lptoken.functions.balanceOf(user).call()
    print('get lp', post_lp - previous_lp)
```

We can get roughly 4x more lp in the first case
## Tools Used
None
## Recommended Mitigation Steps

The current implementation uses `self.balances`

https://github.com/code-423n4/2021-11-bootfinance/blob/main/customswap/contracts/SwapUtils.sol#L1231-L1236
```soliditiy
            for (uint256 i = 0; i < self.pooledTokens.length; i++) {
                uint256 idealBalance = v.d1.mul(self.balances[i]).div(v.d0);
                fees[i] = feePerToken
                    .mul(idealBalance.difference(newBalances[i]))
                    .div(FEE_DENOMINATOR);
                self.balances[i] = newBalances[i].sub(
                    fees[i].mul(self.adminFee).div(FEE_DENOMINATOR)
                );
                newBalances[i] = newBalances[i].sub(fees[i]);
            }
```

Replaces `self.balances` with `_xp(self, newBalances)` would be a simple fix.
I consider the team can take balance's weighted pool as a reference. [WeightedMath.sol#L149-L179](https://github.com/balancer-labs/balancer-v2-monorepo/blob/7ff72a23bae6ce0eb5b134953cc7d5b79a19d099/pkg/pool-weighted/contracts/WeightedMath.sol#L149-L179)

