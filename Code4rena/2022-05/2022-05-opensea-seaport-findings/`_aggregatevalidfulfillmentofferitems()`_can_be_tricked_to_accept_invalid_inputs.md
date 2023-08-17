## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [`_aggregateValidFulfillmentOfferItems()` can be tricked to accept invalid inputs](https://github.com/code-423n4/2022-05-opensea-seaport-findings/issues/75) 

# Lines of code

https://github.com/code-423n4/2022-05-opensea-seaport/blob/main/contracts/lib/FulfillmentApplier.sol#L406


# Vulnerability details

## Impact


The `_aggregateValidFulfillmentOfferItems()` function aims to revert on orders with zero value or where a total consideration amount overflows. Internally this is accomplished by having a temporary variable `errorBuffer`, accumulating issues found, and only reverting once all the items are processed in case there was a problem found. This code is optimistic for valid inputs.

Note: there is a similar issue in `_aggregateValidFulfillmentConsiderationItems()` , which is reported separately.

The problem lies in how this `errorBuffer` is updated:
```solidity
                // Update error buffer (1 = zero amount, 2 = overflow).
                errorBuffer := or(
                  errorBuffer,
                  or(
                    shl(1, lt(newAmount, amount)),
                    iszero(mload(amountPtr))
                  )
                )
```

The final error handling code:
```solidity
            // Determine if an error code is contained in the error buffer.
            switch errorBuffer
            case 1 {
                // Store the MissingItemAmount error signature.
                mstore(0, MissingItemAmount_error_signature)

                // Return, supplying MissingItemAmount signature.
                revert(0, MissingItemAmount_error_len)
            }
            case 2 {
                // If the sum overflowed, panic.
                throwOverflow()
            }
```

While the expected value is `0` (success),  `1` or `2` (failure), it is possible to set it to `3`, which is unhandled and considered as a "success". This can be easily accomplished by having both an overflowing item and a zero item in the order list.

This validation error could lead to fulfilling an order with a consideration (potentially ~0) lower than expected.

## Proof of Concept

Craft an offer containing two errors (e.g. with  zero amount and overflow).
Call `matchOrders()`. Via calls to `_matchAdvancedOrders()`, `_fulfillAdvancedOrders()`, `_applyFulfillment()`, `_aggregateValidFulfillmentOfferItems()` will be called.
The `errorBuffer` will get a value of 3  (the `or` of 1 and 2).
As the value of 3 is not detected, no error will be thrown and the order will be executed, including the mal formed values.

## Tools Used

Manual review

## Recommended Mitigation Steps

1. Change the check on [FulfillmentApplier.sol#L465](https://github.com/code-423n4/2022-05-opensea-seaport/blob/main/contracts/lib/FulfillmentApplier.sol#L465)  to consider `case 3`.
2. Potential option: Introduce an early abort in case `errorBuffer != 0` on [FulfillmentApplier.sol#L338](https://github.com/code-423n4/2022-05-opensea-seaport/blob/main/contracts/lib/FulfillmentApplier.sol#L338)

