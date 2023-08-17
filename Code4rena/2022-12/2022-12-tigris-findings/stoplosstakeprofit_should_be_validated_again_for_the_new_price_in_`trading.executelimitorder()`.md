## Tags

- bug
- 2 (Med Risk)
- primary issue
- selected for report
- sponsor confirmed
- M-18

# [StopLoss/TakeProfit should be validated again for the new price in `Trading.executeLimitOrder()`](https://github.com/code-423n4/2022-12-tigris-findings/issues/512) 

# Lines of code

https://github.com/code-423n4/2022-12-tigris/blob/b2ebb8ea1def4927a747e7a185174892506540ab/contracts/Trading.sol#L506


# Vulnerability details

## Impact
The open price of a stop order might be changed during execution but it doesn't validate StopLoss/TakeProfit for the changed price.

As a result, the executed market order might be closed immediately and there would be an unexpected loss for users.

## Proof of Concept
As we can see from `executeLimitOrder()`, the open price might be changed to the current price for the stop order.

```solidity
File: 2022-12-tigris\contracts\Trading.sol
480:     function executeLimitOrder(
481:         uint _id, 
482:         PriceData calldata _priceData,
483:         bytes calldata _signature
484:     ) 
485:         external
486:     {
487:         unchecked {
488:             _checkDelay(_id, true);
489:             tradingExtension._checkGas();
490:             if (tradingExtension.paused()) revert TradingPaused();
491:             require(block.timestamp >= limitDelay[_id]);
492:             IPosition.Trade memory trade = position.trades(_id);
493:             uint _fee = _handleOpenFees(trade.asset, trade.margin*trade.leverage/1e18, trade.trader, trade.tigAsset, true);
494:             (uint256 _price, uint256 _spread) = tradingExtension.getVerifiedPrice(trade.asset, _priceData, _signature, 0);
495:             if (trade.orderType == 0) revert("5");
496:             if (_price > trade.price+trade.price*limitOrderPriceRange/DIVISION_CONSTANT || _price < trade.price-trade.price*limitOrderPriceRange/DIVISION_CONSTANT) revert("6"); //LimitNotMet
497:             if (trade.direction && trade.orderType == 1) {
498:                 if (trade.price < _price) revert("6"); //LimitNotMet
499:             } else if (!trade.direction && trade.orderType == 1) {
500:                 if (trade.price > _price) revert("6"); //LimitNotMet
501:             } else if (!trade.direction && trade.orderType == 2) {
502:                 if (trade.price < _price) revert("6"); //LimitNotMet
503:                 trade.price = _price;
504:             } else {
505:                 if (trade.price > _price) revert("6"); //LimitNotMet
506:                 trade.price = _price; //@audit check sl/tp
507:             } 
508:             if(trade.direction) {
509:                 trade.price += trade.price * _spread / DIVISION_CONSTANT;
510:             } else {
511:                 trade.price -= trade.price * _spread / DIVISION_CONSTANT;
512:             }

```

But it doesn't validate sl/tp again for the new price so the order might have an invalid sl/tp.

The new price wouldn't satisfy the sl/tp requirements when the price was changed much from the original price due to the high slippage and the order might be closed immediately by sl or tp in this case.

Originally, the protocol validates stoploss only but I say to validate both of stoploss and takeprofit. (I submitted it as another issue to validate tp as well as sl).

## Tools Used
Manual Review

## Recommended Mitigation Steps
Recommend validating sl/tp for the new `trade.price` in `Trading.executeLimitOrder()`.