## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Withdrawing ETH collateral with max uint256 amount value reverts transaction](https://github.com/code-423n4/2022-05-sturdy-findings/issues/85) 

# Lines of code

https://github.com/code-423n4/2022-05-sturdy/blob/78f51a7a74ebe8adfd055bdbaedfddc05632566f/smart-contracts/GeneralVault.sol#L121-L124


# Vulnerability details

## Impact

Withdrawing ETH collateral via the `withdrawCollateral` function using `type(uint256).max` for the `_amount` parameter reverts the transaction due to `_asset` being the zero-address and `IERC20Detailed(_asset).decimals()` not working for native ETH.

## Proof of Concept

[GeneralVault.sol#L121-L124](https://github.com/code-423n4/2022-05-sturdy/blob/78f51a7a74ebe8adfd055bdbaedfddc05632566f/smart-contracts/GeneralVault.sol#L121-L124)

```solidity
if (_amount == type(uint256).max) {
    uint256 decimal = IERC20Detailed(_asset).decimals(); // @audit-info does not work for native ETH. Transaction reverts
    _amount = _amountToWithdraw.mul(this.pricePerShare()).div(10**decimal);
}
```

## Tools Used

Manual review

## Recommended mitigation steps

Check `_asset` and use hard coded decimal value (`18`) for native ETH

