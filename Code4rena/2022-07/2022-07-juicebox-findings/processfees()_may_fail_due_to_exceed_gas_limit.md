## Tags

- bug
- documentation
- 2 (Med Risk)
- sponsor confirmed
- valid

# [processFees() may fail due to exceed gas limit](https://github.com/code-423n4/2022-07-juicebox-findings/issues/8) 

# Lines of code

https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/828bf2f3e719873daa08081cfa0d0a6deaa5ace5/contracts/abstract/JBPayoutRedemptionPaymentTerminal.sol#L594


# Vulnerability details

## processFees() may fail due to exceed gas limit

https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/828bf2f3e719873daa08081cfa0d0a6deaa5ace5/contracts/abstract/JBPayoutRedemptionPaymentTerminal.sol#L594

### Impact

the function `processFees()` in `JBPayoutRedemptionPaymentTerminal.sol` may fail due to unbounded loop over `_heldFeesOf[_projectId]`

`_heldFeesOf[_projectId]` can get very large due to the function `_takeFeeFrom()` where it pushes fees that should be paid to a specific beneficiary onto the array

https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/828bf2f3e719873daa08081cfa0d0a6deaa5ace5/contracts/abstract/JBPayoutRedemptionPaymentTerminal.sol#L1199

`_heldFeesOf[_projectId]` could get large and cause a DOS condition where no fees can be distributed due to exceed of gas limit

### Proof of Concept

```
    for (uint256 _i = 0; _i < _heldFeeLength; ) {
      // Get the fee amount.
      uint256 _amount = _feeAmount(
        _heldFees[_i].amount,
        _heldFees[_i].fee,
        _heldFees[_i].feeDiscount
      );
```

