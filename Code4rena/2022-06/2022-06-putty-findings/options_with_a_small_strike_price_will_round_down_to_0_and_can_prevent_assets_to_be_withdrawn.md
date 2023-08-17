## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Options with a small strike price will round down to 0 and can prevent assets to be withdrawn](https://github.com/code-423n4/2022-06-putty-findings/issues/283) 

# Lines of code

https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L499-L500


# Vulnerability details

## Impact

Certain ERC-20 tokens do not support zero-value token transfers and revert. Using such a token as a `order.baseAsset` for a rather small option strike and a low protocol fee rate can lead to rounding down to 0 and prevent asset withdrawals for those positions.

## Proof of Concept

[PuttyV2.sol#L499-L500](https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L499-L500)

```solidity
// send the fee to the admin/DAO if fee is greater than 0%
uint256 feeAmount = 0;
if (fee > 0) {
    feeAmount = (order.strike * fee) / 1000;
    ERC20(order.baseAsset).safeTransfer(owner(), feeAmount); // @audit-info zero-value ERC20 token transfers can revert for certain tokens
}
```

Some ERC20 tokens revert for zero-value transfers (e.g. `LEND`). If used as a `order.baseAsset` and a small strike price, the fee token transfer will revert. Hence, assets and the strike can not be withdrawn and remain locked in the contract.

See [Weird ERC20 Tokens - Revert on Zero Value Transfers](https://github.com/d-xo/weird-erc20#revert-on-zero-value-transfers)

**Example:**

- `order.baseAsset` is one of those weird ERC-20 tokens
- `order.strike = 999` (depending on the token decimals, a very small option position)
- `fee = 1` (0.1%)

$((999 * 1) / 1000 = 0.999)$ rounded down to 0 -> zero-value transfer reverting transaction

## Tools Used

Manual review

## Recommended mitigation steps

Add a simple check for zero-value token transfers:

```solidity
// send the fee to the admin/DAO if fee is greater than 0%
uint256 feeAmount = 0;
if (fee > 0) {
    feeAmount = (order.strike * fee) / 1000;

    if (feeAmount > 0) {
        ERC20(order.baseAsset).safeTransfer(owner(), feeAmount);
    }
}
```


