## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- resolved
- sponsor confirmed
- old-submission-method
- selected-for-report

# [`fillAsk()` Allows for `msg.value` to be larger than require locking the excess in the contract](https://github.com/code-423n4/2022-07-golom-findings/issues/75) 

# Lines of code

https://github.com/code-423n4/2022-07-golom/blob/e5efa8f9d6dda92a90b8b2c4902320acf0c26816/contracts/core/GolomTrader.sol#L217


# Vulnerability details

## Impact

It is possible to send a higher `msg.value` than is required to `fillAsk()`. The excess value that is sent will be permanently locked in the contract.

## Proof of Concept

There is only one check over `msg.value` and it is that it's greater than `o.totalAmt * amount + p.paymentAmt`. As seen in the following code snippet from #217.
```solidity
        require(msg.value >= o.totalAmt * amount + p.paymentAmt, 'mgmtm');
```

The issue here is that the contract will only ever spend exactly `o.totalAmt * amount + p.paymentAmt`. Hence if `msg.value` is greater than this then the excess value will be permanently locked in the contract.

## Recommended Mitigation Steps

To avoid this issue consider enforcing a strict equality. 

```solidity
        require(msg.value == o.totalAmt * amount + p.paymentAmt, 'mgmtm');
```

