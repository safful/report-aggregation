## Tags

- bug
- help wanted
- disagree with severity
- QA (Quality Assurance)
- sponsor confirmed

# [`cancel()` function does not check if the order already was filled at some point.](https://github.com/code-423n4/2022-06-putty-findings/issues/396) 

# Lines of code

https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L526


# Vulnerability details

## Impact
An **order** could be canceled even after the **order** was filled. Even if this does not affect any other part of the process, the mapping `cancelledOrders` still gets updated and a `CancelledOrder` event is emitted, this could cause issues on a front-end or monitoring tools working with the protocol.

## Proof of Concept

```solidity
function cancel(Order memory order) public {
        require(msg.sender == order.maker, "Not your order");

        bytes32 orderHash = hashOrder(order);

        // mark the order as cancelled
        cancelledOrders[orderHash] = true;

        emit CancelledOrder(orderHash, order);
    }
```

## Recommended Mitigation Steps
Check if the order was already filled before. This could be done by checking if an `nft` with the order id was created before.

```diff
function cancel(Order memory order) public {
        require(msg.sender == order.maker, "Not your order");

        bytes32 orderHash = hashOrder(order);
        
+       require(ownerOf(uint256(orderHash)) == address(0), "This order was already filled");

        // mark the order as cancelled
        cancelledOrders[orderHash] = true;

        emit CancelledOrder(orderHash, order);
    }
```

