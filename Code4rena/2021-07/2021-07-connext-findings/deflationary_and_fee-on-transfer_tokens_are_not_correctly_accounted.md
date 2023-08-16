## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Deflationary and fee-on-transfer tokens are not correctly accounted](https://github.com/code-423n4/2021-07-connext-findings/issues/68) 

# Handle

shw


# Vulnerability details

## Impact

When a router adds liquidity to the `TransactionManager`, the manager does not correctly handle the received amount if the transferred token is a deflationary or fee-on-transfer token. The actual received amount is less than that is recorded in the `routerBalances` variable.

## Proof of Concept

Referenced code:
[TransactionManager.sol#L97](https://github.com/code-423n4/2021-07-connext/blob/main/contracts/TransactionManager.sol#L97)
[TransactionManager.sol#L101](https://github.com/code-423n4/2021-07-connext/blob/main/contracts/TransactionManager.sol#L101)

## Recommended Mitigation Steps

Get the received token amount by calculating the difference of token balance before and after the transfer, for example:

```solidity
uint256 balanceBefore = getOwnBalance(assetId);
require(LibERC20.transferFrom(assetId, router, address(this), amount, "addLiquidity: ERC20_TRANSFER_FAILED");
uint256 receivedAmount = getOwnBalance(assetId) - balanceBefore;

// Update the router balances
routerBalances[router][assetId] += receivedAmount;
```

