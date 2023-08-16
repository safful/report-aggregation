## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Bonding.sol _unbondAndBreak does not account for edge case where no tokens are returned](https://github.com/code-423n4/2021-11-malt-findings/issues/234) 

# Handle

harleythedog


# Vulnerability details

## Impact
In Bonding.sol, the internal function `_unbondAndBreak` transfers a user's stake tokens to the dexHandler and then calls `removeLiquidity` on the dexHandler. Within the Uniswap handler (which is the only handler so far) `removeLiquidity` takes special care in the edge case where `router.removeLiquidity` returns zero tokens. Specifically, the Uniswap handler has this code:
```
if (amountMalt == 0 || amountReward == 0) {
	liquidityBalance = lpToken.balanceOf(address(this));
	lpToken.safeTransfer(msg.sender, liquidityBalance);
	return (amountMalt, amountReward);
}
```

If this edge case does indeed happen (i.e. if something is preventing the Uniswap router from removing liquidity at the moment), then the Uniswap handler will transfer the LP tokens back to Bonding.sol. However, Bonding.sol does not have any logic to recognize that this happened, so the LP tokens will become stuck in the contract and the user will never get any of their value back. This could be very bad if the user unbonds a lot of LP and they don't get any of it back.

## Proof of Concept
See `_unbondAndBreak` here: https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/Bonding.sol#L226

Notice how the edge case where `amountMalt == 0 || amountReward == 0` is not considered in this function, but it is considered in the Uniswap handler's `removeLiquidity` here: https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/DexHandlers/UniswapHandler.sol#L240

## Tools Used
Inspection.

## Recommended Mitigation Steps
Add a similar edge case check to `_unbondAndBreak`. In the case where LP tokens are transferred back to Bonding.sol instead of malt/reward, these LP tokens should be forwarded back to the user since the value is rightfully theirs.

