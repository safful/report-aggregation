## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Unnecessary `SLOAD`s in `Basket`](https://github.com/code-423n4/2021-10-defiprotocol-findings/issues/34) 

# Handle

pants


# Vulnerability details

The functions `Basket.handleFees()`, `Basket.changePublisher()`, `Basket.changeLicenseFee()`, `Basket.publishNewIndex()`, `Basket.deleteNewIndex()`, `Basket.updateIBRatio()`, `Basket.approveUnderlying()`, `Basket.pushUnderlying()` and `Basket.pullUnderlying()` read values from storage multiple times instead of caching them in local variables:
- `Basket.handleFees()` reads `lastFee` up to twice, `factory` 3 times and `ibRatio` once (when `newIbRatio` can be used).
- `Basket.changePublisher()` reads `pendingPublisher.publisher` up to twice and `publisher` up to once (when `newPublisher` can be used).
- `Basket.changeLicenseFee()` reads `pendingLicenseFee.licenseFee` up to twice and `licenseFee` up to once (when `newLicenseFee` can be used).
- `Basket.publishNewIndex()` reads `auction` up to 3 times and `licenseFee` up to once (when `newLicenseFee` can be used).
- `Basket.deleteNewIndex()` reads `publisher` twice and `auction` up to twice.
- `Basket.updateIBRatio()` reads `ibRatio` twice.
- `Basket.approveUnderlying()` reads `tokens[i]` twice.
- `Basket.pushUnderlying()` reads `ibRatio` once per iteration.
- `Basket.pullUnderlying()` reads `ibRatio` once per iteration.

## Impact
Storage reads are much more expensive than reading local variables.

## Tool Used
Manual code review.

## Recommended Mitigation Steps
Read these values from storage once, cache them in local variables and then read them again from the local variables.

