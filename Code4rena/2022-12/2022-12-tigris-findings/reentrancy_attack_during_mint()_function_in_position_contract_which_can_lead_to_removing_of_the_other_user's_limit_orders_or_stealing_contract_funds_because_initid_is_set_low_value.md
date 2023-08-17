## Tags

- bug
- 3 (High Risk)
- primary issue
- selected for report
- sponsor confirmed
- H-07

# [reentrancy attack during mint() function in Position contract which can lead to removing of the other user's limit orders or stealing contract funds because initId is set low value](https://github.com/code-423n4/2022-12-tigris-findings/issues/400) 

# Lines of code

https://github.com/code-423n4/2022-12-tigris/blob/588c84b7bb354d20cbca6034544c4faa46e6a80e/contracts/Position.sol#L126-L161


# Vulnerability details

## Impact
Function `Position.mint()` has been used in `initiateLimitOrder()` and `initiateMarketOrder()` and it doesn't follow check-effect-interaction pattern and code updates the values of `_limitOrders`, `initId`, `_openPositions` and `position _tokenIds` variables after making external call by using `safeMint()`. This would give attacker opportunity to reenter the Trading contract logics and perform malicious action while contract storage state is wrong. the only limitation of the attacker is that he need to bypass `_checkDelay()` checks. attacker can perform this action:
1. call `initiateLimitOrder()` and create limit order with id equal to ID1 reenter (while `_limitOrders` for ID1 is not yet settled) with `cancelLimitOrder(ID1)` (no `checkDelay()` check) and remove other users limit orders because code would try to remove `_limitOrderIndexes[_asset][ID1]` position but the value is 0 and code would remove limit order in the index 0 which belongs to another user in the `Position.burn()` code.
2. call `initiateMarketOrder()` and create a position with ID1 and while `initId[ID1]` has not yet settled reenter the Trading with `addToPosition(ID1)` function (bypass `checkDelay()` because both action is opening) and increase the position size which would set `initId[ID1]` according to new position values but then when code execution returns to rest of `mint()` logic `initId[ID1]` would set by initial values of the positions which is very lower than what it should be and `initId[ID1]` has been used for calculating `accuredInterest` of the position which is calculated for profit and loss of position and contract would calculate more profit for position and would pay attacker more profit from contract balances.

## Proof of Concept
This is `mint()` code in Position contract:
```
    function mint(
        MintTrade memory _mintTrade
    ) external onlyMinter {
        uint newTokenID = _tokenIds.current();

        Trade storage newTrade = _trades[newTokenID];
        newTrade.margin = _mintTrade.margin;
        newTrade.leverage = _mintTrade.leverage;
        newTrade.asset = _mintTrade.asset;
        newTrade.direction = _mintTrade.direction;
        newTrade.price = _mintTrade.price;
        newTrade.tpPrice = _mintTrade.tp;
        newTrade.slPrice = _mintTrade.sl;
        newTrade.orderType = _mintTrade.orderType;
        newTrade.id = newTokenID;
        newTrade.tigAsset = _mintTrade.tigAsset;

        _safeMint(_mintTrade.account, newTokenID);   // make external call because of safeMint() usage
        if (_mintTrade.orderType > 0) { // update the values of some storage functions
            _limitOrders[_mintTrade.asset].push(newTokenID);
            _limitOrderIndexes[_mintTrade.asset][newTokenID] = _limitOrders[_mintTrade.asset].length-1;
        } else {
            initId[newTokenID] = accInterestPerOi[_mintTrade.asset][_mintTrade.tigAsset][_mintTrade.direction]*int256(_mintTrade.margin*_mintTrade.leverage/1e18)/1e18;
            _openPositions.push(newTokenID);
            _openPositionsIndexes[newTokenID] = _openPositions.length-1;

            _assetOpenPositions[_mintTrade.asset].push(newTokenID);
            _assetOpenPositionsIndexes[_mintTrade.asset][newTokenID] = _assetOpenPositions[_mintTrade.asset].length-1;
        }
        _tokenIds.increment();
    }
```
As you can see by calling `_safeMint()` code would make external call to `onERC721Received()` function of the account address and the code sets the values for `_limitOrders[]`, `_limitOrderIndexes[]`, `initId[]`, `_openPositions[]`, `_openPositionsIndexes[]`, `_assetOpenPositions[]`, `_assetOpenPositionsIndexes[]` and `_tokenIds`. so code don't follow check-effect-interaction pattern and it's possible to perform reentrancy attack.
there could be multiple scenarios that attacker can perform the attack and do some damage. two of them are:


**scenario #1 where attacker remove other users limit orders and create broken storage state**
1. attacker contract would call `initiateLimitOrder()` and code would create the limit order and mint it in the `Position._safeMint()` with ID1.
2. then code would call attacker address in `_safeMint()` function because of the `onERC721Received()` call check.
3. variables `_limitOrders[]`, `_limitOrderIndexes[ID1]` are not yet updated for ID1 and `_limitOrderIndexes[ID1]` is 0x0 and ID1 is not in `_limitOrder[]` list.
4. attacker contract would reenter the Trading contract by calling `cancelLimitOrder(ID1)`.
5. `cancelLimitOrder()` checks would pass and would tries to call `Position.burn(ID1)`.
6. `burn()` function would tries to remove ID1 from `_limitOrders[]` list but because `_limitOrderIndexes[ID1]` is 0 so code would remove the 0 index limit order which is belongs to another user.
7. execution would return to `Position.mint()` logic and code would add burned id token to `_limitOrder[]` list.

so there is two impact here, first other users limit order got removed and the second is that contract storage had bad state and burned tokens get stock in the list.


**scenario #2 where attacker steal contract/users funds by wrong profit calculation**
1. attacker's contract would call `initiateMarketOrder(lowMargin)` to create position with ID1 while the margin is low.
2. code would mint position token for attacker and in `_safeMint()` would make external call and call `onERC721Received()` function of attacker address.
3. the value of `initId[ID1]` is not yet set for ID1.
4. attacker contract would call `addToPosition(ID1, bigMargin)` to increase the margin of the position the `_checkDelay()` check would pass because both actions are opening position.
5. code would increase the margin of the position and set the value of the `initId[ID1]` by calling `position.addToPosition()` and the value were be based on the `newMargin`.
6. the execution flow would receive the rest of `Position.mint()` function and code would set `initId[ID1]` based on old margin value.
7. then the value of `initId[ID1]` for attacker position would be very low which would cause `accInterest` to be very higher than it supposed to be for position(in `Position.trades()` function calculations ) and would cause `_payout` value to be very high (in `pnl()` function's calculations) and when attacker close position ID1 attacker would receive a lot more profit from it.

so attacker created a position with a lot of profit by reentering the logics and manipulating calculation of the profits for the position.

there can be other scenarios possible to perform and damage the protocol or users because there is no reentrancy protection mechanism and attacker only need to bypass validity checks of functions.

## Tools Used
VIM

## Recommended Mitigation Steps
follow the check-effect-interaction pattern.