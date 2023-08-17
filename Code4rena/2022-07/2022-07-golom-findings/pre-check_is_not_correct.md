## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- resolved
- sponsor confirmed
- selected-for-report

# [Pre-check is not correct](https://github.com/code-423n4/2022-07-golom-findings/issues/851) 

# Lines of code

https://github.com/code-423n4/2022-07-golom/blob/e5efa8f9d6dda92a90b8b2c4902320acf0c26816/contracts/core/GolomTrader.sol#L342
https://github.com/code-423n4/2022-07-golom/blob/e5efa8f9d6dda92a90b8b2c4902320acf0c26816/contracts/core/GolomTrader.sol#L397


# Vulnerability details

## Impact
`fillCriteriaBid` can be reverted due to the pre-check while it can work.

## Proof of Concept
When `refererrAmt > 0` and `referrer` address is not set (is 0), 
`(o.totalAmt - protocolfee - o.exchange.paymentAmt - o.prePayment.paymentAmt) * amount - p.paymentAmt >= 0` and `o.totalAmt < o.exchange.paymentAmt + o.prePayment.paymentAmt + o.refererrAmt` can hold true at the same time.

It is when `o.refererrAmt > (p.paymentAmt + protocolfee) / amount`.
In that case, `_settleBalances` can work, but fillCriteriaBid will be reverted due to the check in line 342.


## Tools Used
Manual review

## Recommended Mitigation Steps
I think `require(o.totalAmt >= o.exchange.paymentAmt + o.prePayment.paymentAmt)` is correct.