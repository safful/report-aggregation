## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Missing Complication check in `takeMultipleOneOrders`](https://github.com/code-423n4/2022-06-infinity-findings/issues/125) 

# Lines of code

https://github.com/code-423n4/2022-06-infinity/blob/765376fa238bbccd8b1e2e12897c91098c7e5ac6/contracts/core/InfinityExchange.sol#L300-L328


# Vulnerability details

An order's type and it's rules are defined in it's `Complication`. Not checking it would allow anyone to take any orders regardless of their Complication's rule, causing unexpected execution for order makers.

`takeMultipleOneOrders` assumes that all `makerOrders` are simple orderbook orders and the  Complication check is missing here.

#### Proof of Concept
- Alice signs a makerOrder with `PrivateSaleComplication`, allowing only Bob to take the private sale order.
- A malicious trader calls `takeMultipleOneOrders` to take Alice's order, despite the Complication only allowing Bob to take it.

#### Recommended Mitigation Steps
Add `canExecTakeOneOrder` function in IComplication.sol and implement it in `InfinityOrderBookComplication` (and future Complications) to support `takeMultipleOneOrders` operation, then modify `takeMultipleOneOrders` to use the check:
```
function takeMultipleOneOrders() {
    ...
    for (uint256 i = 0; i < numMakerOrders; ) {
        bytes32 makerOrderHash = _hash(makerOrders[i]);
        bool makerOrderValid = isOrderValid(makerOrders[i], makerOrderHash);
        bool executionValid = IComplication(makerOrders[i].execParams[0]).canExecTakeOneOrder(makerOrders[i]);
        
        require(makerOrderValid && executionValid, 'order not verified');
        
        require(currency == makerOrders[i].execParams[1], 'cannot mix currencies');
        require(isMakerSeller == makerOrders[i].isSellOrder, 'cannot mix order sides');
        uint256 execPrice = _getCurrentPrice(makerOrders[i]);
        totalPrice += execPrice; // @audit-issue missing complication check
        _execTakeOneOrder(makerOrderHash, makerOrders[i], isMakerSeller, execPrice);
        unchecked {
            ++i;
        }
    }
    ...
}
```

