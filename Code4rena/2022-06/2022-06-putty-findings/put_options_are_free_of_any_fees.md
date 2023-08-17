## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Put options are free of any fees](https://github.com/code-423n4/2022-06-putty-findings/issues/285) 

# Lines of code

https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L450-L451


# Vulnerability details

## Impact

Fees are expected to be paid whenever an option is exercised (as per the function comment on [L235](https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L235)).

### Put options

If a put option is exercised, the exerciser receives the strike price (initially deposited by the short position holder) denominated in `order.baseAsset`.

### Call options

If a call option is exercised, the exerciser sends the strike price to Putty and the short position holder is able to withdraw the strike amount.

However, the current protocol implementation is missing to deduct fees for exercised put options. Put options are free of any fees.

## Proof of Concept

The protocol fee is correctly charged for exercised calls:

[PuttyV2.withdraw](https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L494-L506)

```solidity
// transfer strike to owner if put is expired or call is exercised
if ((order.isCall && isExercised) || (!order.isCall && !isExercised)) {
    // send the fee to the admin/DAO if fee is greater than 0%
    uint256 feeAmount = 0;
    if (fee > 0) {
        feeAmount = (order.strike * fee) / 1000;
        ERC20(order.baseAsset).safeTransfer(owner(), feeAmount); // @audit DoS due to reverting erc20 token transfer (weird erc20 tokens, blacklisted or paused owner; erc777 hook on owner receiver side can prevent transfer hence reverting and preventing withdrawal) - use pull pattern @high  // @audit zero value token transfers can revert. Small strike prices and low fee can lead to rounding down to 0 - check feeAmount > 0 @high  // @audit should not take fees if renounced owner (zero address) as fees can not be withdrawn @medium
    }

    ERC20(order.baseAsset).safeTransfer(msg.sender, order.strike - feeAmount); // @audit fee should not be paid if strike is simply returned to short owner for expired put @high

    return;
}
```

Contrary, put options are free of any fees:

[PuttyV2.sol#L450-L451](https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L450-L451)

```solidity
// transfer strike from putty to exerciser
ERC20(order.baseAsset).safeTransfer(msg.sender, order.strike);
```

## Tools Used

Manual review

## Recommended mitigation steps

Charge fees also for exercised put options.


