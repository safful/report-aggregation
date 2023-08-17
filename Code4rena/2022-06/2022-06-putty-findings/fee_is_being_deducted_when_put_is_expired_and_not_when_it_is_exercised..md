## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed

# [Fee is being deducted when Put is expired and not when it is exercised.](https://github.com/code-423n4/2022-06-putty-findings/issues/269) 

# Lines of code

https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L495-L503
https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L451


# Vulnerability details

## Impact
Fee is being deducted when Put is expired and not when it is exercised in `PuttyV2.sol`.
Comment section of the `setFee()` function mentions `"fee rate that is applied on exercise"` which signifies that the fee amount is meant to be deducted from strike only when a position is being exercised (or has been exercised).

But, in function `withdraw()` at [PuttyV2.solL#495-L503](https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L495-L503)  the fee is being deducted even when the Put position is not exercised and has expired. 

Also, in function `exercise()` there is no fee deduction from the `order.strike` when the Put position is exercised and the strike is being transferred to the caller ([PuttyV2.solL#451](https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L451)).

This unintended deduction from assets of Put Shorter and the absence of fee deduction from strike when Put is exercised are directly impacting the assets and therefore marked as Medium Risk.

## Proof of Concept
`if` condition present at [PuttyV2.solL#495](https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L495) passes if `order.isCall` is `false` and `isExercised` is false.

`feeAmount` becomes positive if `fee > 0` and it gets deducted from the `order.strike` which gets transferred to `msg.sender` at line number [PuttyV2.solL#503](https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L503).

## Tools Used
Manual Analysis

## Recommended Mitigation Steps
1. Update `if` condition at [PuttyV2.sol#L498](https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L498) with `(fee > 0 && order.isCall && isExercised)`

2. Add feeAmount calculation and deduction after put is exercised and strike is transferred at [PuttyV2.sol#L451](https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L451) as follows:

```solidity
uint256 feeAmount = 0;
if (fee > 0) {
    feeAmount = (order.strike * fee) / 1000;
    ERC20(order.baseAsset).safeTransfer(owner(), feeAmount);
}
ERC20(order.baseAsset).safeTransfer(msg.sender, order.strike - feeAmount);
```

