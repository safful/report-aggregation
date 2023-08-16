## Tags

- bug
- 2 (Med Risk)
- SwappableYieldSource
- sponsor confirmed

# [Inconsistent balance when supplying transfer-on-fee or deflationary tokens](https://github.com/code-423n4/2021-07-pooltogether-findings/issues/52) 

# Handle

shw


# Vulnerability details

## Impact

The `supplyTokenTo` function of `SwappableYieldSource` assumes that `amount` of `_depositToken` is transferred to itself after calling the `safeTransferFrom` function (and thus it supplies `amount` of token to the yield source). However, this may not be true if the `_depositToken` is a transfer-on-fee token or a deflationary/rebasing token, causing the received amount to be less than the accounted amount.

## Proof of Concept

Referenced code:
[SwappableYieldSource.sol#L211-L212](https://github.com/pooltogether/swappable-yield-source/blob/89cf66a3e3f8df24a082e1cd0a0e80d08953049c/contracts/SwappableYieldSource.sol#L211-L212)

## Recommended Mitigation Steps

Get the actual received amount by calculating the difference of token balance before and after the transfer. For example, re-writing line 211-212 to:

```solidity
uint256 balanceBefore = _depositToken.balanceOf(address(this));
_depositToken.safeTransferFrom(msg.sender, address(this), amount);
uint256 receivedAmount = _depositToken.balanceOf(address(this)) - balanceBefore;
yieldSource.supplyTokenTo(receivedAmount, address(this));
```

