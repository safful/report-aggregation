## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Maker buy order with no specified NFT tokenIds may get fulfilled in `matchOneToManyOrders` without receiving any NFT](https://github.com/code-423n4/2022-06-infinity-findings/issues/254) 

# Lines of code

https://github.com/code-423n4/2022-06-infinity/blob/765376fa238bbccd8b1e2e12897c91098c7e5ac6/contracts/core/InfinityOrderBookComplication.sol#L68-L116


# Vulnerability details

The call stack: matchOneToManyOrders() -> _matchOneMakerSellToManyMakerBuys() -> _execMatchOneMakerSellToManyMakerBuys() -> _execMatchOneToManyOrders() -> _transferMultipleNFTs()

Based on the context, a maker buy order can set `OrderItem.tokens` as an empty array to indicate that they can accept any tokenId in this collection, in that case, `InfinityOrderBookComplication.doTokenIdsIntersect()` will always return `true`.

However, when the system matching a sell order with many buy orders, the `InfinityOrderBookComplication` contract only ensures that the specified tokenIds intersect with the sell order, and the total count of specified tokenIds equals the sell order's quantity (`makerOrder.constraints[0]`).

This allows any maker buy order with same collection and `empty tokenIds` to be added to `manyMakerOrders` as long as there is another maker buy order with specified tokenIds that matched the sell order's tokenIds.

https://github.com/code-423n4/2022-06-infinity/blob/765376fa238bbccd8b1e2e12897c91098c7e5ac6/contracts/core/InfinityOrderBookComplication.sol#L68-L116

```solidity
function canExecMatchOneToMany(
    OrderTypes.MakerOrder calldata makerOrder,
    OrderTypes.MakerOrder[] calldata manyMakerOrders
  ) external view override returns (bool) {
    uint256 numItems;
    bool isOrdersTimeValid = true;
    bool itemsIntersect = true;
    uint256 ordersLength = manyMakerOrders.length;
    for (uint256 i = 0; i < ordersLength; ) {
      if (!isOrdersTimeValid || !itemsIntersect) {
        return false; // short circuit
      }

      uint256 nftsLength = manyMakerOrders[i].nfts.length;
      for (uint256 j = 0; j < nftsLength; ) {
        numItems += manyMakerOrders[i].nfts[j].tokens.length;
        unchecked {
          ++j;
        }
      }

      isOrdersTimeValid =
        isOrdersTimeValid &&
        manyMakerOrders[i].constraints[3] <= block.timestamp &&
        manyMakerOrders[i].constraints[4] >= block.timestamp;

      itemsIntersect = itemsIntersect && doItemsIntersect(makerOrder.nfts, manyMakerOrders[i].nfts);

      unchecked {
        ++i;
      }
    }

    bool _isTimeValid = isOrdersTimeValid &&
      makerOrder.constraints[3] <= block.timestamp &&
      makerOrder.constraints[4] >= block.timestamp;

    uint256 currentMakerOrderPrice = _getCurrentPrice(makerOrder);
    uint256 sumCurrentOrderPrices = _sumCurrentPrices(manyMakerOrders);

    bool _isPriceValid = false;
    if (makerOrder.isSellOrder) {
      _isPriceValid = sumCurrentOrderPrices >= currentMakerOrderPrice;
    } else {
      _isPriceValid = sumCurrentOrderPrices <= currentMakerOrderPrice;
    }

    return (numItems == makerOrder.constraints[0]) && _isTimeValid && itemsIntersect && _isPriceValid;
  }
```

However, because `buy.nfts` is used as `OrderItem` to transfer the nfts from seller to buyer, and there are no tokenIds specified in the matched maker buy order, the buyer wont receive any nft (`_transferERC721s` does nothing, 0 transfers) despite the buyer paid full in price.

https://github.com/code-423n4/2022-06-infinity/blob/765376fa238bbccd8b1e2e12897c91098c7e5ac6/contracts/core/InfinityExchange.sol#L763-L786

```solidity
function _execMatchOneMakerSellToManyMakerBuys(
    bytes32 sellOrderHash,
    bytes32 buyOrderHash,
    OrderTypes.MakerOrder calldata sell,
    OrderTypes.MakerOrder calldata buy,
    uint256 startGasPerOrder,
    uint256 execPrice,
    uint16 protocolFeeBps,
    uint32 wethTransferGasUnits,
    address weth
  ) internal {
    isUserOrderNonceExecutedOrCancelled[buy.signer][buy.constraints[5]] = true;
    uint256 protocolFee = (protocolFeeBps * execPrice) / 10000;
    uint256 remainingAmount = execPrice - protocolFee;
    _execMatchOneToManyOrders(sell.signer, buy.signer, buy.nfts, buy.execParams[1], remainingAmount);
    _emitMatchEvent(
      sellOrderHash,
      buyOrderHash,
      sell.signer,
      buy.signer,
      buy.execParams[0],
      buy.execParams[1],
      execPrice
    );
```

https://github.com/code-423n4/2022-06-infinity/blob/765376fa238bbccd8b1e2e12897c91098c7e5ac6/contracts/core/InfinityExchange.sol#L1080-L1092

```solidity
function _transferERC721s(
    address from,
    address to,
    OrderTypes.OrderItem calldata item
  ) internal {
    uint256 numTokens = item.tokens.length;
    for (uint256 i = 0; i < numTokens; ) {
      IERC721(item.collection).safeTransferFrom(from, to, item.tokens[i].tokenId);
      unchecked {
        ++i;
      }
    }
  }
```


### PoC

1. Alice signed and submitted a maker buy order #1, to buy `2` Punk with `2 WETH` and specified tokenIds = `1`,`2`
2. Bob signed and submitted a maker buy order #2, to buy `1` Punk with `1 WETH` and with no specified tokenIds.
3. Charlie signed and submitted a maker sell order #3, ask for `3 WETH` for `2` Punk and specified tokenIds = `1`,`2`
4. The match executor called `matchOneToManyOrders()` match Charlie's sell order #3 with buy order #1 and #2, Alice received `2` Punk, Charlie received `3 WETH`, Bob paid `1 WETH` and get nothing in return.

### Recommendation

Change to:

```solidity
function canExecMatchOneToMany(
    OrderTypes.MakerOrder calldata makerOrder,
    OrderTypes.MakerOrder[] calldata manyMakerOrders
  ) external view override returns (bool) {
    uint256 numItems;
    uint256 numConstructedItems;
    bool isOrdersTimeValid = true;
    bool itemsIntersect = true;
    uint256 ordersLength = manyMakerOrders.length;
    for (uint256 i = 0; i < ordersLength; ) {
      if (!isOrdersTimeValid || !itemsIntersect) {
        return false; // short circuit
      }

      numConstructedItems += manyMakerOrders[i].constraints[0];

      uint256 nftsLength = manyMakerOrders[i].nfts.length;
      for (uint256 j = 0; j < nftsLength; ) {
        numItems += manyMakerOrders[i].nfts[j].tokens.length;
        unchecked {
          ++j;
        }
      }

      isOrdersTimeValid =
        isOrdersTimeValid &&
        manyMakerOrders[i].constraints[3] <= block.timestamp &&
        manyMakerOrders[i].constraints[4] >= block.timestamp;

      itemsIntersect = itemsIntersect && doItemsIntersect(makerOrder.nfts, manyMakerOrders[i].nfts);

      unchecked {
        ++i;
      }
    }

    bool _isTimeValid = isOrdersTimeValid &&
      makerOrder.constraints[3] <= block.timestamp &&
      makerOrder.constraints[4] >= block.timestamp;

    uint256 currentMakerOrderPrice = _getCurrentPrice(makerOrder);
    uint256 sumCurrentOrderPrices = _sumCurrentPrices(manyMakerOrders);

    bool _isPriceValid = false;
    if (makerOrder.isSellOrder) {
      _isPriceValid = sumCurrentOrderPrices >= currentMakerOrderPrice;
    } else {
      _isPriceValid = sumCurrentOrderPrices <= currentMakerOrderPrice;
    }

    return (numItems == makerOrder.constraints[0]) && (numConstructedItems == numItems) && _isTimeValid && itemsIntersect && _isPriceValid;
  }
```

