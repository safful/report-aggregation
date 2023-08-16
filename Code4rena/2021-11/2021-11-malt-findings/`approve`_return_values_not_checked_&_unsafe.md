## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [`approve` return values not checked & unsafe](https://github.com/code-423n4/2021-11-malt-findings/issues/247) 

# Handle

cmichel


# Vulnerability details

The `ERC20.approve()` function returns a boolean value indicating success.
This parameter needs to be checked for success.
Some tokens do **not** revert if the transfer failed but return `false` instead.

In addition, some tokens (like [USDT L199](https://etherscan.io/address/0xdac17f958d2ee523a2206206994597c13d831ec7#code)) do not work when changing the allowance from an existing non-zero allowance value.
They must first be approved by zero and then the actual allowance must be approved.

```solidity
IERC20(token).safeApprove(address(operator), 0);
IERC20(token).safeApprove(address(operator), amount);
```

This issue exists for example in `AuctionParticipant.purchaseArbitrageTokens`:

```solidity
auctionRewardToken.approve(address(auction), balance);
```

As well as in `UniswapHandler.buyMalt`:

```solidity
rewardToken.approve(address(router), rewardBalance);
```

## Impact
Tokens that don't correctly implement the latest EIP20 spec, by either returning `false` on failure or reverting if approved from a non-zero value, will be unusable in the protocol as they revert the transaction because of the missing return value.

## Recommended Mitigation Steps
We recommend using OpenZeppelin’s `SafeERC20` versions with the `safeApprove(0)` functions that handle the return value check as well as non-standard-compliant tokens.

