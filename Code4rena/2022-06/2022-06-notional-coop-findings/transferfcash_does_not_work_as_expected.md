## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- Notional

# [transferfCash does not work as expected](https://github.com/code-423n4/2022-06-notional-coop-findings/issues/98) 

# Lines of code

https://github.com/code-423n4/2022-06-notional-coop/blob/main/notional-wrapped-fcash/contracts/wfCashLogic.sol#L184


# Vulnerability details

## Impact
If maturity is reached and user has asked for redeem with opts.transferfCash as true, then if (hasMatured()) turns true at wfCashLogic.sol#L216 causing fcash to be cashed out in underlying token and then sent to receiver. So receiver obtains underlying when fcash was expected. The sender wont get an error thinking fcash transfer was success

## Proof of Concept

1. User A calls redeem with opts.transferfCash as true and receiver as User B
2. Since maturity is reached so instead of transferring the fCash, contract would simply cash out fCash and sent the underlying token to the receiver which was not expected

## Recommended Mitigation Steps
If opts.transferfCash is true and maturity is reached then throw an error mentioning that fCash can no longer be transferred

