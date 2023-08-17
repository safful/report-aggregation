## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed

# [`acceptCounterOffer()` May Result In Both Orders Being Filled](https://github.com/code-423n4/2022-06-putty-findings/issues/44) 

# Lines of code

https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L573-L584


# Vulnerability details

## Impact

When a user is attempting to accept a counter offer they call the function [acceptCounterOffer()](https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L573-L584) with both the `originalOrder` to be cancelled and the new `order` to fill. It is possible for an attacker (or any other user who happens to call `fillOrder()` at the same time) to fill the `originalOrder` before `acceptCounterOffer()` cancels it.

The impact is that both `originalOrder` and `order` are filled. The `msg.sender` of `acceptCounterOffer()` is twice as leveraged as they intended to be if the required token transfers succeed.

## Proof of Concept

[acceptCounterOffer()](https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L573-L584) calls `cancel()` on the original order, however it will not revert if the order has already been filled.
```solidity
    function acceptCounterOffer(
        Order memory order,
        bytes calldata signature,
        Order memory originalOrder
    ) public payable returns (uint256 positionId) {
        // cancel the original order
        cancel(originalOrder);


        // accept the counter offer
        uint256[] memory floorAssetTokenIds = new uint256[](0);
        positionId = fillOrder(order, signature, floorAssetTokenIds);
    }
```

[cancel()](https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L526-L535) does not revert if an order has already been filled it only prevents future `fillOrder()` transactions from succeeding.
```solidity
    function cancel(Order memory order) public {
        require(msg.sender == order.maker, "Not your order");


        bytes32 orderHash = hashOrder(order);


        // mark the order as cancelled
        cancelledOrders[orderHash] = true;


        emit CancelledOrder(orderHash, order);
    }
```

Therefore any user may front-run the `acceptCounterOffer()` transaction with a `fillOrder()` transaction that fills the original order. As a result the user ends up filling both `order` and `originalOrder`. Then `acceptCounterOffer()` cancels the `originalOrder` which is essentially a no-op since it's been filled and continues to fill the new `order` resulting in both orders being filled.

## Recommended Mitigation Steps

Consider having `cancel()` revert if an order has already been filled. This can be done by adding the following line `require(_ownerOf[uint256(orderHash)] == 0)`.

