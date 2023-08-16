## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [[WP-G11] Check of `_amount > 0` can be done earlier to save gas](https://github.com/code-423n4/2022-01-insure-findings/issues/230) 

# Handle

WatchPug


# Vulnerability details

When there are multiple checks, adjusting the sequence to allow the tx to fail earlier can save some gas.

Checks using less gas should be executed earlier than those with higher gas costs, to avoid unnecessary storage read, arithmetic operations, etc when it reverts.

For example:

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/CDSTemplate.sol#L238-L256

```solidity
require(paused == false, "ERROR: PAUSED");
require(
    request.timestamp +
        parameters.getLockup(msg.sender) <
        block.timestamp,
    "ERROR: WITHDRAWAL_QUEUE"
);
require(
    request.timestamp +
        parameters.getLockup(msg.sender) +
        parameters.getWithdrawable(msg.sender) >
        block.timestamp,
    "ERROR: WITHDRAWAL_NO_ACTIVE_REQUEST"
);
require(
    request.amount >= _amount,
    "ERROR: WITHDRAWAL_EXCEEDED_REQUEST"
);
require(_amount > 0, "ERROR: WITHDRAWAL_ZERO");
```

The check of `_amount > 0` can be done earlier to avoid reading from storage when `_amount = 0`.

## Recommendation

Change to:

```solidity
        require(paused == false, "ERROR: PAUSED");
        require(_amount > 0, "ERROR: WITHDRAWAL_ZERO");
        require(
            request.amount >= _amount,
            "ERROR: WITHDRAWAL_EXCEEDED_REQUEST"
        );
        require(
            request.timestamp +
                parameters.getLockup(msg.sender) <
                block.timestamp,
            "ERROR: WITHDRAWAL_QUEUE"
        );
        require(
            request.timestamp +
                parameters.getLockup(msg.sender) +
                parameters.getWithdrawable(msg.sender) >
                block.timestamp,
            "ERROR: WITHDRAWAL_NO_ACTIVE_REQUEST"
        );
```

Other examples include:

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/CDSTemplate.sol#L189-L191

```solidity
uint256 _balance = balanceOf(msg.sender);
require(_balance >= _amount, "ERROR: REQUEST_EXCEED_BALANCE");
require(_amount > 0, "ERROR: REQUEST_ZERO");
```

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/IndexTemplate.sol#L217-L236

```solidity
require(locked == false, "ERROR: WITHDRAWAL_PENDING");
require(
    _requestTime + _lockup < block.timestamp,
    "ERROR: WITHDRAWAL_QUEUE"
);
require(
    _requestTime + _lockup + parameters.getWithdrawable(msg.sender) >
        block.timestamp,
    "ERROR: WITHDRAWAL_NO_ACTIVE_REQUEST"
);
require(
    withdrawalReq[msg.sender].amount >= _amount,
    "ERROR: WITHDRAWAL_EXCEEDED_REQUEST"
);
require(_amount > 0, "ERROR: WITHDRAWAL_ZERO");

require(
    _retVal <= withdrawable(),
    "ERROR: WITHDRAW_INSUFFICIENT_LIQUIDITY"
);
```

