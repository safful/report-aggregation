## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [[WP-H17] Users will lose a majority or even all of the rewards when the amount of total shares is too large, due to precision loss](https://github.com/code-423n4/2022-03-biconomy-findings/issues/140) 

# Lines of code

https://github.com/code-423n4/2022-03-biconomy/blob/db8a1fdddd02e8cc209a4c73ffbb3de210e4a81a/contracts/hyphen/LiquidityFarming.sol#L265-L291


# Vulnerability details

https://github.com/code-423n4/2022-03-biconomy/blob/db8a1fdddd02e8cc209a4c73ffbb3de210e4a81a/contracts/hyphen/LiquidityFarming.sol#L265-L291

```solidity
function getUpdatedAccTokenPerShare(address _baseToken) public view returns (uint256) {
    uint256 accumulator = 0;
    uint256 lastUpdatedTime = poolInfo[_baseToken].lastRewardTime;
    uint256 counter = block.timestamp;
    uint256 i = rewardRateLog[_baseToken].length - 1;
    while (true) {
        if (lastUpdatedTime >= counter) {
            break;
        }
        unchecked {
            accumulator +=
                rewardRateLog[_baseToken][i].rewardsPerSecond *
                (counter - max(lastUpdatedTime, rewardRateLog[_baseToken][i].timestamp));
        }
        counter = rewardRateLog[_baseToken][i].timestamp;
        if (i == 0) {
            break;
        }
        --i;
    }

    // We know that during all the periods that were included in the current iterations,
    // the value of totalSharesStaked[_baseToken] would not have changed, as we only consider the
    // updates to the pool that happened after the lastUpdatedTime.
    accumulator = (accumulator * ACC_TOKEN_PRECISION) / totalSharesStaked[_baseToken];
    return accumulator + poolInfo[_baseToken].accTokenPerShare;
}
```

https://github.com/code-423n4/2022-03-biconomy/blob/04751283f85c9fc94fb644ff2b489ec339cd9ffc/contracts/hyphen/LiquidityProviders.sol#L286-L292

```solidity
uint256 mintedSharesAmount;
// Adding liquidity in the pool for the first time
if (totalReserve[token] == 0) {
    mintedSharesAmount = BASE_DIVISOR * _amount;
} else {
    mintedSharesAmount = (_amount * totalSharesMinted[token]) / totalReserve[token];
}
```

In `HyphenLiquidityFarming`, the `accTokenPerShare` is calculated based on the total staked shares.

However, as the `mintedSharesAmount` can easily become very large on `LiquidityProviders.sol`, all the users can lose their rewards due to precision loss.

### PoC

Given:

- rewardsPerSecond is `10e18`;
- lastRewardTime is 24 hrs ago;

Then:

1. Alice `addTokenLiquidity()` with `1e8 * 1e18` XYZ on B-Chain, totalSharesMinted == `1e44`;
2. Alice `deposit()` to HyphenLiquidityFarming, totalSharesStaked == `1e44`;
3. 24 hrs later, Alice tries to claim the rewards.


`accumulator = rewardsPerSecond * 24 hours` == 864000e18 == 8.64e23

Expected Results: As the sole staker, Alice should get all the `864000e18` rewards.

Actual Results: Alice received 0 rewards.

That's becasue when `totalSharesStaked > 1e36`, `accumulator = (accumulator * ACC_TOKEN_PRECISION) / totalSharesStaked[_baseToken];` will be round down to `0`.

When the `totalSharesStaked` is large enough, all users will lose their rewards due to precision loss.

### Recommendation

1. Consider lowering the `BASE_DIVISOR` so that the initial share price can be higher;
2. Consider making `ACC_TOKEN_PRECISION` larger to prevent precision loss;

See also the Recommendation on [WP-H14].

