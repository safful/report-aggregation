## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Incentive Pool can be drained without rebalancing the pool](https://github.com/code-423n4/2022-03-biconomy-findings/issues/87) 

# Lines of code

https://github.com/code-423n4/2022-03-biconomy/blob/04751283f85c9fc94fb644ff2b489ec339cd9ffc/contracts/hyphen/LiquidityPool.sol#L149-L173
https://github.com/code-423n4/2022-03-biconomy/blob/04751283f85c9fc94fb644ff2b489ec339cd9ffc/contracts/hyphen/LiquidityPool.sol#L263-L277


# Vulnerability details

## Impact
`depositErc20` allows an attacker to specify the destination chain to be the same as the source chain and the receiver account to be the same as the caller account. This enables an attacker to drain the incentive pool without rebalancing the pool back to the equilibrium state. 


## Proof of Concept
This requires the attacker to have some collateral, to begin with. The profit also depends on how much the attacker has. Assume the attacker has enough assets.

In each chain, when the pool is very deficit (e.g. `currentLiquidity` is much less than `providedLiquidity`), which often mean there's a good amount in the Incentive pool after some high valued transfers, then do the following.  

- step 1 :  borrow the liquidityDifference amount such that one can get the whole incentivePool.
```
            uint256 liquidityDifference = providedLiquidity - currentLiquidity;
            if (amount >= liquidityDifference) {
                rewardAmount = incentivePool[tokenAddress];
```
- step 2 : call `depositErc20()` with `toChainId` being the same chain and `receiver` being `msg.sender`.

The executor will call `sendFundsToUser` to msg.sender. Then a rewardAmount, equivalent to the entire incentive pool (up to 10% of the total pool value), will be added to `msg.sender` minus equilibrium fee (~0.01%) and gas fee. 


In the end, the pool is back to the deficit state as before, the incentive pool is drained and the exploiter pockets the difference of rewardAmount minus fees. 

This attack can be repeated on each deployed chain multiple times whenever the incentive pool is profitable (particularly right after a big transfer).



## Tools Used

## Recommended Mitigation Steps
- Disallow `toChainId` to be the source chain by validating it in `depositErc20` or in `sendFundsToUser` validate that `fromChainId` is not the same as current chain.

- require `receiver` is not `msg.sender` in `depositErc20`. 




