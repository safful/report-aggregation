## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [YieldManagerAave.sol: Wrong branch in depositPaymentToken() if amountReservedInCaseOfInsufficientAaveLiquidity == amount](https://github.com/code-423n4/2021-08-floatcapital-findings/issues/74) 

# Handle

hickuphh3


# Vulnerability details

### Impact

In the unlikely event `amountReservedInCaseOfInsufficientAaveLiquidity == amount`, the `else` case will be executed, which means `lendingPool.deposit()` is called with a value of zero. It would therefore be better to change the condition so that the `if` case is executed instead.

### Recommended Mitigation Steps

```jsx
function depositPaymentToken(uint256 amount) external override longShortOnly {
  // If amountReservedInCaseOfInsufficientAaveLiquidity isn't zero, then efficiently net the difference between the amount
  // It basically always be zero besides extreme and unlikely situations with aave.
  if (amountReservedInCaseOfInsufficientAaveLiquidity != 0) {
		// instead of strictly greater than
    if (amountReservedInCaseOfInsufficientAaveLiquidity >= amount) {
      amountReservedInCaseOfInsufficientAaveLiquidity -= amount;
      // Return early, nothing to deposit into the lending pool
      return;
    }
	...
}
```

