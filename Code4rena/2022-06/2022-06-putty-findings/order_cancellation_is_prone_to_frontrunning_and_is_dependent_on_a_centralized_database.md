## Tags

- bug
- help wanted
- 2 (Med Risk)
- resolved
- sponsor confirmed
- old-submission-method

# [Order cancellation is prone to frontrunning and is dependent on a centralized database](https://github.com/code-423n4/2022-06-putty-findings/issues/186) 

# Lines of code

https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L526-L535


# Vulnerability details

## Impact

Order cancellation requires makers to call `cancel()`, inputting the order as a function parameter. This is the only cancellation method, and it can cause two issues.

This first issue is that it is an on-chain signal for MEV users to frontrun the cancellation and fill the order.

The second issue is the dependency to a centralized service for cancelling the order. As orders are signed off chain, they would be stored in a centralized database. It is unlikely that an end user would locally record all the orders they make. This means that when cancelling an order, maker needs to request the order parameters from the centralized service. If the centralized service goes offline, it could allow malicious parties who have a copy of the order database to fill orders that would have been cancelled otherwise.

## Proof of Concept

1. Bob signs an order which gets recorded in Putty servers.
2. Alice mirrors all the orders using Putty APIs.
3. Putty servers go offline.
4. Bob wants to cancel his order because changing token prices makes his order less favourable to him.
5. Bob cannot cancel his order because Putty servers are down and he does not remember the exact amounts of tokens he used.
6. Alice goes through all the orders in her local mirror and fulfills the non-cancelled orders, including Bob's, with extremely favourable terms for herself.

## Tools Used

Pen & paper.

## Recommended Mitigation Steps

Aside from the standard order cancellation method, have an extra method to cancel all orders of a caller. This can be achieved using a "minimum valid nonce" state variable, as a mapping from user address to nonce.

```solidity
mapping(address => uint256) minimumValidNonce;
```

Allow users to increment their `minimumValidNonce`. Make sure the incrementation function do not allow incrementing more than `2**64` such that callers cannot lock themselves out of creating orders by increasing `minimumValidNonce` to `2**256-1` by mistake. Then, prevent filling orders if `order.nonce < minimumValidNonce`.

Another method to achieve bulk cancelling is using counters. For example, Seaport [uses counters](https://github.com/ProjectOpenSea/seaport/blob/171f2cd7faf13b2bf0455851499f1981274977f7/contracts/lib/CounterManager.sol), which is an extra order parameter that has to match the corresponding counter state variable. It allows maker to cancel all his orders by [incrementing the counter state variable by one](https://github.com/ProjectOpenSea/seaport/blob/171f2cd7faf13b2bf0455851499f1981274977f7/contracts/lib/Consideration.sol#L475-L478). 

Either of these extra cancellation methods would enable cancelling orders without signalling to MEV bots, and without a dependency to a centralized database.

