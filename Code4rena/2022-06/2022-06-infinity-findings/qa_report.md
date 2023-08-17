## Tags

- bug
- QA (Quality Assurance)
- sponsor confirmed

# [QA Report](https://github.com/code-423n4/2022-06-infinity-findings/issues/232) 

# Summary

We list 2 low-critical findings:
* (Low) Misunderstanding params
* (Low) `takeMultipleOneOrders` doesn’t check nfts length

# (Low) Misunderstanding params

## Impact

`constraints` in struct MakerOrder is an array which is easily misused.

## Proof of Concept

It hard coded a number to indicate an element in `constraints` array:

https://github.com/code-423n4/2022-06-infinity/blob/main/contracts/core/InfinityExchange.sol#L515-L516

```
    bool orderExpired = isUserOrderNonceExecutedOrCancelled[order.signer][order.constraints[5]] ||
      order.constraints[5] < userMinOrderNonce[order.signer];
```

## Tools Used

None

## Recommended Mitigation Steps

Consider to define variables rather than `constraints` array, or use constraint index to indicate:

```
    enum CONSTRAINT_INDEX {
        numItems,
        startPrice,
        endPrice,
        startTime,
        endTime,
        nonce
    }
```

# (Low) `takeMultipleOneOrders` doesn’t check nfts length

## Impact

In `matchOneToOneOrders`, it checks that the `nfts` of orders must be 1:
https://github.com/code-423n4/2022-06-infinity/blob/main/contracts/core/InfinityOrderBookComplication.sol#L32-L37

But in `takeMultipleOneOrders`, it doesn’t check that the `nfts` of orders must be 1, and the comment says: Batch buys or sells orders with specific `1` NFTs.

## Tools Used

None

## Recommended Mitigation Steps

Also check nfts length in `​​takeMultipleOneOrders`.
