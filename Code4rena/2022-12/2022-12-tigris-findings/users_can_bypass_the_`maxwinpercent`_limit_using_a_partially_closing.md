## Tags

- bug
- 3 (High Risk)
- disagree with severity
- judge review requested
- primary issue
- selected for report
- sponsor confirmed
- H-09

# [Users can bypass the `maxWinPercent` limit using a partially closing](https://github.com/code-423n4/2022-12-tigris-findings/issues/507) 

# Lines of code

https://github.com/code-423n4/2022-12-tigris/blob/b2ebb8ea1def4927a747e7a185174892506540ab/contracts/Trading.sol#L625-L627


# Vulnerability details

## Impact
Users can bypass the `maxWinPercent` limit using a partial closing.

As a result, users can receive more funds than their upper limit from the protocol.

## Proof of Concept
As we can see from the [documentation](https://docs.tigris.trade/protocol/trading-and-fees#limitations), there is limitation of a maximum PnL.

```
Maximum PnL is +500%. The trade won't be closed unless the user sets a Take Profit order or closes the position manually.
```

And this logic was implemented like below in `_closePosition()`.

```solidity
File: 2022-12-tigris\contracts\Trading.sol
624:                 _toMint = _handleCloseFees(_trade.asset, uint256(_payout)*_percent/DIVISION_CONSTANT, _trade.tigAsset, _positionSize*_percent/DIVISION_CONSTANT, _trade.trader, _isBot);
625:                 if (maxWinPercent > 0 && _toMint > _trade.margin*maxWinPercent/DIVISION_CONSTANT) { //@audit bypass limit
626:                     _toMint = _trade.margin*maxWinPercent/DIVISION_CONSTANT;
627:                 }
```

But it checks the `maxWinPercent` between the partial payout and full margin so the below scenario is possible.

1. Alice opened an order of margin = 100 and PnL = 1000 after taking closing fees.
2. If `maxWinPercent` = 500%, Alice should receive 500 at most.
3. But Alice closed 50% of the position and she got 500 for a 50% margin because it checks `maxWinPercent` with `_toMint = 500` and `_trade.margin = 100`
4. After she closed 50% of the position, the remaining margin = 50 and PnL = 500 so she can continue step 3 again and again.
5. As a result, she can withdraw almost 100% of the initial PnL(1000) even though she should receive at most 500.

## Tools Used
Manual Review

## Recommended Mitigation Steps
We should check the `maxWinPercent` between the partial payout and partial margin like below.

```solidity
    _toMint = _handleCloseFees(_trade.asset, uint256(_payout)*_percent/DIVISION_CONSTANT, _trade.tigAsset, _positionSize*_percent/DIVISION_CONSTANT, _trade.trader, _isBot);

    uint256 partialMarginToClose = _trade.margin * _percent / DIVISION_CONSTANT; //+++++++++++++++++++++++
    if (maxWinPercent > 0 && _toMint > partialMarginToClose*maxWinPercent/DIVISION_CONSTANT) { 
        _toMint = partialMarginToClose*maxWinPercent/DIVISION_CONSTANT;
    }
```