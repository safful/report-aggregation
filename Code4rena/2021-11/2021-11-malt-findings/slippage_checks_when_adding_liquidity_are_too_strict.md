## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Slippage checks when adding liquidity are too strict](https://github.com/code-423n4/2021-11-malt-findings/issues/257) 

# Handle

cmichel


# Vulnerability details

When adding liquidity through `UniswapHandler.addLiquidity`, the entire contract balances are used to add liquidity and the min amounts are set to 95% of these balances.
If the balances in this contract are unbalanced (the ratio is not similar to the current Uniswap pool reserve ratios) then this function will revert and no liquidity is added.

See `UniswapHandler.buyMalt`:

```solidity
(maltUsed, rewardUsed, liquidityCreated) = router.addLiquidity(
  address(malt),
  address(rewardToken),
  maltBalance, // @audit-info amountADesired
  rewardBalance,
  // @audit assumes that whatever is in this contract is already balanced. good assumption?
  maltBalance.mul(95).div(100), // @audit-info amountAMin
  rewardBalance.mul(95).div(100),
  msg.sender, // transfer LP tokens to sender
  now
);
```

## Impact
If the contract has unbalanced balances, then the `router.addLiquidity` call will revert.
Note that an attacker could even send tokens to this contract to make them unbalanced and revert, resulting in a griefing attack.

## Recommended Mitigation Steps
It needs to be ensured that the balances in the contract are always balanced and match the current reserve ratio.
It might be better to avoid directly using the balances which can be manipulated by transferring tokens to the contract and accepting parameters instead of how many tokens to provide liquidity with from the caller side.

