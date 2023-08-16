## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas Optimizations](https://github.com/code-423n4/2022-02-hubble-findings/issues/135) 

# Gas Report

**Table of Contents:**

- [Gas Report](#gas-report)
  - [Foreword](#foreword)
  - [Summary](#summary)
  - [File: AMM.sol](#file-ammsol)
    - [function initialize()](#function-initialize)
      - [Use `calldata` instead of `memory` for `string _name`](#use-calldata-instead-of-memory-for-string-_name)
    - [function openPosition()](#function-openposition)
      - [Do not cache `positions[trader]` in memory](#do-not-cache-positionstrader-in-memory)
    - [function liquidatePosition()](#function-liquidateposition)
      - [Do not cache `positions[trader]` in memory](#do-not-cache-positionstrader-in-memory-1)
    - [function removeLiquidity()](#function-removeliquidity)
      - [Do not cache `positions[maker]` in memory](#do-not-cache-positionsmaker-in-memory)
      - [Do not cache `makers[maker]` in memory](#do-not-cache-makersmaker-in-memory)
    - [function getNotionalPositionAndUnrealizedPnl()](#function-getnotionalpositionandunrealizedpnl)
      - [Do not cache `positions[trader]` in memory](#do-not-cache-positionstrader-in-memory-2)
      - [Do not cache `makers[trader]` in memory](#do-not-cache-makerstrader-in-memory)
    - [function getPendingFundingPayment()](#function-getpendingfundingpayment)
      - [Do not cache `positions[trader]` in memory](#do-not-cache-positionstrader-in-memory-3)
      - [Do not cache `makers[trader]` in memory](#do-not-cache-makerstrader-in-memory-1)
    - [function getTakerNotionalPositionAndUnrealizedPnl()](#function-gettakernotionalpositionandunrealizedpnl)
      - [Do not cache `positions[trader]` in memory](#do-not-cache-positionstrader-in-memory-4)
    - [function _emitPositionChanged()](#function-_emitpositionchanged)
      - [Do not cache `positions[trader]` in memory](#do-not-cache-positionstrader-in-memory-5)
    - [function _openReversePosition()](#function-_openreverseposition)
      - [Do not cache `positions[trader]` in memory](#do-not-cache-positionstrader-in-memory-6)
      - [Unchecked block L597](#unchecked-block-l597)
    - [function _calcTwap()](#function-_calctwap)
      - [Do not cache `reserveSnapshots[snapshotIndex]` in memory](#do-not-cache-reservesnapshotssnapshotindex-in-memory)
      - [Cache `reserveSnapshots.length` in memory](#cache-reservesnapshotslength-in-memory)
      - [Unchecked block L684](#unchecked-block-l684)
      - [Use the cache for calculation](#use-the-cache-for-calculation)
  - [File: ClearingHouse.sol](#file-clearinghousesol)
    - [function _disperseLiquidationFee()](#function-_disperseliquidationfee)
      - [Unchecked block L214](#unchecked-block-l214)
  - [File: InsuranceFund.sol](#file-insurancefundsol)
    - [function pricePerShare()](#function-pricepershare)
      - [Unchecked block L97](#unchecked-block-l97)
  - [File: Oracle.sol](#file-oraclesol)
    - [function getUnderlyingTwapPrice()](#function-getunderlyingtwapprice)
      - [Unchecked block L81](#unchecked-block-l81)
  - [File: Interfaces.sol](#file-interfacessol)
    - [struct Collateral](#struct-collateral)
      - [Tight packing structs to save slots](#tight-packing-structs-to-save-slots)
  - [File: MarginAccount.sol](#file-marginaccountsol)
    - [function _getLiquidationInfo()](#function-_getliquidationinfo)
      - [Do not cache `supportedCollateral[idx]` in memory](#do-not-cache-supportedcollateralidx-in-memory)
    - [function _transferOutVusd()](#function-_transferoutvusd)
      - [Unchecked block L588](#unchecked-block-l588)
  - [File: VUSD.sol](#file-vusdsol)
    - [function processWithdrawals()](#function-processwithdrawals)
      - [Unchecked block L57-L65](#unchecked-block-l57-l65)
      - [Cache `start` in memory](#cache-start-in-memory)
  - [General recommendations](#general-recommendations)
    - [Variables](#variables)
      - [No need to explicitly initialize variables with default values](#no-need-to-explicitly-initialize-variables-with-default-values)
      - [Pre-increments cost less gas compared to post-increments](#pre-increments-cost-less-gas-compared-to-post-increments)
    - [Comparisons](#comparisons)
      - [`> 0` is less efficient than `!= 0` for unsigned integers (with proof)](#-0-is-less-efficient-than--0-for-unsigned-integers-with-proof)
    - [For-Loops](#for-loops)
      - [An array's length should be cached to save gas in for-loops](#an-arrays-length-should-be-cached-to-save-gas-in-for-loops)
      - [`++i` costs less gas compared to `i++`](#i-costs-less-gas-compared-to-i)
      - [Increments can be unchecked](#increments-can-be-unchecked)
    - [Arithmetics](#arithmetics)
      - [Shift Right instead of Dividing by 2](#shift-right-instead-of-dividing-by-2)
    - [Errors](#errors)
      - [Reduce the size of error messages (Long revert Strings)](#reduce-the-size-of-error-messages-long-revert-strings)
      - [Use Custom Errors instead of Revert Strings to save Gas](#use-custom-errors-instead-of-revert-strings-to-save-gas)

## Foreword

- **Storage-reading optimizations**

> The code can be optimized by minimising the number of SLOADs. SLOADs are expensive (100 gas) compared to MLOADs/MSTOREs (3 gas). In the paragraphs below, please see the `@audit-issue` tags in the pieces of code's comments for more information about SLOADs that could be saved by caching the mentioned **storage** variables in **memory** variables.

- **Unchecking arithmetics operations that can't underflow/overflow**

> Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn't possible (as an example, when a comparison is made before the arithmetic operation, or the operation doesn't depend on user input), some gas can be saved by using an `unchecked` block: <https://docs.soliditylang.org/en/v0.8.10/control-structures.html#checked-or-unchecked-arithmetic>

- **`@audit` tags**

> The code is annotated at multiple places with `//@audit` comments to pinpoint the issues. Please, pay attention to them for more details.

## Summary

- One pattern that was often seen is caching structs in memory when it's not needed. A copy in memory of a storage struct will trigger as many SLOADs as there are slots. If the struct's fields are only read once, or if the number of storage reading would be inferior to the number of slots: don't cache the struct in memory.

## File: AMM.sol

### function initialize()

```
093:     function initialize(
094:         address _registry,
095:         address _underlyingAsset,
096:         string memory _name,//@audit readonly: calldata
097:         address _vamm,
098:         address _governance
099:     ) external initializer {
100:         _setGovernace(_governance);
101: 
102:         vamm = IVAMM(_vamm);
103:         underlyingAsset = _underlyingAsset;
104:         name = _name;
105:         fundingBufferPeriod = 15 minutes;
106: 
107:         syncDeps(_registry);
108:     }
```

#### Use `calldata` instead of `memory` for `string _name`

An external function passing a readonly variable should mark it as `calldata` and not `memory`

### function openPosition()

```
113:     function openPosition(address trader, int256 baseAssetQuantity, uint quoteAssetLimit)
114:         override
115:         external
116:         onlyClearingHouse
117:         returns (int realizedPnl, uint quoteAsset, bool isPositionIncreased)
118:     {
119:         require(ammState == AMMState.Active, "AMM.openPosition.not_active");
120:         Position memory position = positions[trader]; //@audit 3 SLOADs vs 1 enough
121:         bool isNewPosition = position.size == 0 ? true : false;
122:         Side side = baseAssetQuantity > 0 ? Side.LONG : Side.SHORT;
123:         if (isNewPosition || (position.size > 0 ? Side.LONG : Side.SHORT) == side) {
124:             // realizedPnl = 0;
125:             quoteAsset = _increasePosition(trader, baseAssetQuantity, quoteAssetLimit);
126:             isPositionIncreased = true;
127:         } else {
128:             (realizedPnl, quoteAsset, isPositionIncreased) = _openReversePosition(trader, baseAssetQuantity, quoteAssetLimit);
129:         }
130:         _emitPositionChanged(trader, realizedPnl);
131:     }
```

#### Do not cache `positions[trader]` in memory

As a copy in memory of a struct makes as many SLOADs as there are slots, here a copy costs 3 SLOADs:

```
41:     struct Position {
42:         int256 size;
43:         uint256 openNotional;
44:         int256 lastPremiumFraction;
45:     }
```

However, only the `size` field is read twice. Therefore, only this field should get cached: `int256 _size = positions[trader].size;`

### function liquidatePosition()

```
133:     function liquidatePosition(address trader)
134:         override
135:         external
136:         onlyClearingHouse
137:         returns (int realizedPnl, uint quoteAsset)
138:     {
139:         // don't need an ammState check because there should be no active positions
140:         Position memory position = positions[trader]; //@audit 3 SLOADs vs 1 enough
141:         bool isLongPosition = position.size > 0 ? true : false;
142:         // sending market orders can fk the trader. @todo put some safe guards around price of liquidations
143:         if (isLongPosition) {
144:             (realizedPnl, quoteAsset) = _reducePosition(trader, -position.size, 0);
145:         } else {
146:             (realizedPnl, quoteAsset) = _reducePosition(trader, -position.size, type(uint).max);
147:         }
148:         _emitPositionChanged(trader, realizedPnl);
149:     }
```

#### Do not cache `positions[trader]` in memory

Similar to [Do not cache `positions[trader]` in memory](#do-not-cache-positionstrader-in-memory).

However, only the `size` field is read 3 times. Therefore, only this field should get cached: `int256 _size = positions[trader].size;`

### function removeLiquidity()

```
133:     function liquidatePosition(address trader)
134:         override
135:         external
136:         onlyClearingHouse
137:         returns (int realizedPnl, uint quoteAsset)
138:     {
139:         // don't need an ammState check because there should be no active positions
140:         Position memory position = positions[trader]; //@audit 3 SLOADs vs 1 enough
141:         bool isLongPosition = position.size > 0 ? true : false;
142:         // sending market orders can fk the trader. @todo put some safe guards around price of liquidations
143:         if (isLongPosition) {
144:             (realizedPnl, quoteAsset) = _reducePosition(trader, -position.size, 0);
145:         } else {
146:             (realizedPnl, quoteAsset) = _reducePosition(trader, -position.size, type(uint).max);
147:         }
148:         _emitPositionChanged(trader, realizedPnl);
149:     }
```

#### Do not cache `positions[maker]` in memory

Similar to [Do not cache `positions[trader]` in memory](#do-not-cache-positionstrader-in-memory).
However, here, even the fields shouldn't get cached, as they are read only once:

```
220:         Position memory _taker = positions[maker];//@audit 3 SLOADs vs 2 enough
...
233:             _taker.size,
234:             _taker.openNotional
```

Therefore, use `220:         Position storage _taker = positions[maker];`

#### Do not cache `makers[maker]` in memory

Similarly, a copy in memory for `Maker` costs 7 SLOADs:

```
48:     struct Maker {
49:         uint vUSD;
50:         uint vAsset;
51:         uint dToken;
52:         int pos; // position
53:         int posAccumulator; // value of global.posAccumulator until which pos has been updated
54:         int lastPremiumFraction;
55:         int lastPremiumPerDtoken;
56:     }
```

Here, caching the first 5 fields in memory is enough.

### function getNotionalPositionAndUnrealizedPnl()

```
395:     function getNotionalPositionAndUnrealizedPnl(address trader)
396:         override
397:         external
398:         view
399:         returns(uint256 notionalPosition, int256 unrealizedPnl, int256 size, uint256 openNotional)
400:     {
401:         Position memory _taker = positions[trader];//@audit 3 SLOADs vs 2 enough
402:         Maker memory _maker = makers[trader];//@audit 7 SLOADs vs 3 enough
403: 
404:         (notionalPosition, size, unrealizedPnl, openNotional) = vamm.get_notional(
405:             _maker.dToken,
406:             _maker.vUSD,
407:             _maker.vAsset,
408:             _taker.size,
409:             _taker.openNotional
410:         );
411:     }
```

#### Do not cache `positions[trader]` in memory

Here, we need `Position storage _taker = positions[trader];`

#### Do not cache `makers[trader]` in memory

Here, we need `Maker storage _maker = makers[trader];`

### function getPendingFundingPayment()

```
425:         Position memory taker = positions[trader];//@audit 3 SLOADs vs 2 enough
...
434:         Maker memory maker = makers[trader];//@audit 7 SLOADs vs 5 enough
```

#### Do not cache `positions[trader]` in memory

Here, we need `Position storage _taker = positions[trader];`

#### Do not cache `makers[trader]` in memory

Here, we need `Maker storage _maker = makers[trader];`

### function getTakerNotionalPositionAndUnrealizedPnl()

```
458:     function getTakerNotionalPositionAndUnrealizedPnl(address trader) override public view returns(uint takerNotionalPosition, int256 unrealizedPnl) {
459:         Position memory position = positions[trader];//@audit 3 SLOADs vs 2 enough
460:         if (position.size > 0) {
461:             takerNotionalPosition = vamm.get_dy(1, 0, position.size.toUint256());
462:             unrealizedPnl = takerNotionalPosition.toInt256() - position.openNotional.toInt256();
463:         } else if (position.size < 0) {
464:             takerNotionalPosition = vamm.get_dx(0, 1, (-position.size).toUint256());
465:             unrealizedPnl = position.openNotional.toInt256() - takerNotionalPosition.toInt256();
466:         }
467:     }
```

#### Do not cache `positions[trader]` in memory

Here, we need to cache these fields: `size` and `openNotional`

### function _emitPositionChanged()

```
527:     function _emitPositionChanged(address trader, int256 realizedPnl) internal {
528:         Position memory position = positions[trader];//@audit 3 SLOADs vs 2 enough
529:         emit PositionChanged(trader, position.size, position.openNotional, realizedPnl);
530:     }
```

#### Do not cache `positions[trader]` in memory

Here, we need `Position storage _taker = positions[trader];`

### function _openReversePosition()

```
584:     function _openReversePosition(address trader, int256 baseAssetQuantity, uint quoteAssetLimit)
585:         internal
586:         returns (int realizedPnl, uint quoteAsset, bool isPositionIncreased)
587:     {
588:         Position memory position = positions[trader];//@audit 3 SLOADs vs 1 enough
589:         if (abs(position.size) >= abs(baseAssetQuantity)) {
590:             (realizedPnl, quoteAsset) = _reducePosition(trader, baseAssetQuantity, quoteAssetLimit);
591:         } else {
592:             uint closedRatio = (quoteAssetLimit * abs(position.size).toUint256()) / abs(baseAssetQuantity).toUint256();
593:             (realizedPnl, quoteAsset) = _reducePosition(trader, -position.size, closedRatio);
594: 
595:             // this is required because the user might pass a very less value (slippage-prone) while shorting
596:             if (quoteAssetLimit >= quoteAsset) {
597:                 quoteAssetLimit -= quoteAsset; //@audit uncheck (see L596)
598:             }
599:             quoteAsset += _increasePosition(trader, baseAssetQuantity + position.size, quoteAssetLimit);
600:             isPositionIncreased = true;
601:         }
602:     }
```

#### Do not cache `positions[trader]` in memory

Here, we need to cache the `size` field

#### Unchecked block L597

This line can't underflow due to the condition L596. Therefore, it should be wrapped in an `unchecked` block

### function _calcTwap()

```
656:     function _calcTwap(uint256 _intervalInSeconds)
657:         internal
658:         view
659:         returns (uint256)
660:     {
661:         uint256 snapshotIndex = reserveSnapshots.length - 1; //@audit reserveSnapshots.length  SLOAD 1
662:         uint256 currentPrice = reserveSnapshots[snapshotIndex].lastPrice;
663:         if (_intervalInSeconds == 0) {
664:             return currentPrice;
665:         }
666: 
667:         uint256 baseTimestamp = _blockTimestamp() - _intervalInSeconds;
668:         ReserveSnapshot memory currentSnapshot = reserveSnapshots[snapshotIndex];//@audit 3 SLOADs vs 1 enough
669:         // return the latest snapshot price directly
670:         // if only one snapshot or the timestamp of latest snapshot is earlier than asking for
671:         if (reserveSnapshots.length == 1 || currentSnapshot.timestamp <= baseTimestamp) {//@audit reserveSnapshots.length  SLOAD 2
...
675:         uint256 previousTimestamp = currentSnapshot.timestamp;
676:         uint256 period = _blockTimestamp() - previousTimestamp;
677:         uint256 weightedPrice = currentPrice * period;
678:         while (true) {
...
680:             if (snapshotIndex == 0) {
681:                 return weightedPrice / period;
682:             }
...
684:             snapshotIndex = snapshotIndex - 1; //@audit uncheck (see L680-L682)
685:             currentSnapshot = reserveSnapshots[snapshotIndex];
686:             currentPrice = reserveSnapshots[snapshotIndex].lastPrice; //@audit use currentSnapshot.lastPrice
...
689:             if (currentSnapshot.timestamp <= baseTimestamp) {
...
698:             uint256 timeFraction = previousTimestamp - currentSnapshot.timestamp;
...
701:             previousTimestamp = currentSnapshot.timestamp;
```

#### Do not cache `reserveSnapshots[snapshotIndex]` in memory

Here, we need to cache the `timestamp` field. Copying the struct in memory costs 3 SLOADs.

#### Cache `reserveSnapshots.length` in memory

This would save 1 SLOAD

#### Unchecked block L684

This line can't underflow due to the condition L680-L682. Therefore, it should be wrapped in an `unchecked` block

#### Use the cache for calculation

As we already have `currentSnapshot = reserveSnapshots[snapshotIndex];`: use it here: `currentPrice = currentSnapshot.lastPrice;`

## File: ClearingHouse.sol

### function _disperseLiquidationFee()

```
210:     function _disperseLiquidationFee(uint liquidationFee) internal {
211:         if (liquidationFee > 0) {
212:             uint toInsurance = liquidationFee / 2;
213:             marginAccount.transferOutVusd(address(insuranceFund), toInsurance);
214:             marginAccount.transferOutVusd(_msgSender(), liquidationFee - toInsurance); //@audit uncheck (see L212)
215:         }
216:     }
```

#### Unchecked block L214

This line can't underflow due to the condition L212. Therefore, it should be wrapped in an `unchecked` block

## File: InsuranceFund.sol

### function pricePerShare()

```
File: InsuranceFund.sol
094:     function pricePerShare() external view returns (uint) {
095:         uint _totalSupply = totalSupply();
096:         uint _balance = balance();
097:         _balance -= Math.min(_balance, pendingObligation); //@audit uncheck
098:         if (_totalSupply == 0 || _balance == 0) 
099:             return PRECISION;
100:         }
101:         return _balance * PRECISION / 
_totalSupply;
102:     }
```

#### Unchecked block L97

This line can't underflow for obvious mathematical reasons (`_balance` substracting at most itself). Therefore, it should be wrapped in an `unchecked` block

## File: Oracle.sol

### function getUnderlyingTwapPrice()

#### Unchecked block L81

This line can't underflow due to L76-L79. Therefore, it should be wrapped in an `unchecked` block

## File: Interfaces.sol

### struct Collateral

#### Tight packing structs to save slots

While this file is out of scope, it deeply impacts MarginAccount.sol.
I suggest going from:

```
94:     struct Collateral {
95:         IERC20 token; //@audit 20 bytes
96:         uint weight; //@audit 32 bytes
97:         uint8 decimals; //@audit 1 byte
98:     }
```

to

```
94:     struct Collateral {
95:         IERC20 token; //@audit 20 bytes
96:         uint8 decimals; //@audit 1 byte
97:         uint weight; //@audit 32 bytes
98:     }
```

To save 1 slot per array element in MarginAccount.sol's storage

## File: MarginAccount.sol

### function _getLiquidationInfo()

```
460:     function _getLiquidationInfo(address trader, uint idx) internal view returns (LiquidationBuffer memory buffer) {
461:         require(idx > VUSD_IDX && idx < supportedCollateral.length, "collateral not seizable");
462:         (buffer.status, buffer.repayAble, buffer.incentivePerDollar) = isLiquidatable(trader, false);
463:         if (buffer.status == IMarginAccount.LiquidationStatus.IS_LIQUIDATABLE) {
464:             Collateral memory coll = supportedCollateral[idx];//@audit 3 SLOADs vs 2 enough
465:             buffer.priceCollateral = oracle.getUnderlyingPrice(address(coll.token)).toUint256();
466:             buffer.decimals = coll.decimals;
467:         }
468:     }
```

#### Do not cache `supportedCollateral[idx]` in memory

Here, we need `Collateral storage coll = supportedCollateral[idx];`. Copying the struct in memory costs 3 SLOADs.

### function _transferOutVusd()

#### Unchecked block L588

This line can't underflow due to L583. Therefore, it should be wrapped in an `unchecked` block

## File: VUSD.sol

### function processWithdrawals()

```
53:     function processWithdrawals() external {
54:         uint reserve = reserveToken.balanceOf(address(this));
55:         require(reserve >= withdrawals[start].amount, 'Cannot process withdrawals at this time: Not enough balance'); //@audit start SLOAD 1
56:         uint i = start;//@audit start SLOAD 2
57:         while (i < withdrawals.length && (i - start) <= maxWithdrawalProcesses) { //@audit uncheck whole //@audit start SLOAD 3
58:             Withdrawal memory withdrawal = withdrawals[i]; //@audit-ok
59:             if (reserve < withdrawal.amount) {
60:                 break;
61:             }
62:             reserveToken.safeTransfer(withdrawal.usr, withdrawal.amount);
63:             reserve -= withdrawal.amount;  //@audit uncheck (see L59-L61)
64:             i += 1;
65:         }
66:         start = i;
67:     }
```

#### Unchecked block L57-L65

The whole while-loop can't underflow. Therefore, it should be wrapped in an `unchecked` block

#### Cache `start` in memory

Cache `start` in memory as `initialStart` and use it L55 + L57 (compare `i` to it in the while-loop)

## General recommendations

### Variables

#### No need to explicitly initialize variables with default values

If a variable is not set/initialized, it is assumed to have the default value (`0` for `uint`, `false` for `bool`, `address(0)` for address...). Explicitly initializing it with its default value is an anti-pattern and wastes gas.

As an example: `for (uint256 i = 0; i < numIterations; ++i) {` should be replaced with `for (uint256 i; i < numIterations; ++i) {`

Instances include:  

```
ClearingHouse.sol:122:        for (uint i = 0; i < amms.length; i++) {
ClearingHouse.sol:130:        for (uint i = 0; i < amms.length; i++) {
ClearingHouse.sol:170:        for (uint i = 0; i < amms.length; i++) {
ClearingHouse.sol:194:        for (uint i = 0; i < amms.length; i++) { // liquidate all positions
ClearingHouse.sol:251:        for (uint i = 0; i < amms.length; i++) {
ClearingHouse.sol:263:        for (uint i = 0; i < amms.length; i++) {
ClearingHouse.sol:277:        for (uint i = 0; i < amms.length; i++) {
InsuranceFund.sol:52:        uint shares = 0;
MarginAccount.sol:31:    uint constant VUSD_IDX = 0;
MarginAccount.sol:331:        for (uint i = 0; i < idxs.length; i++) {
MarginAccount.sol:521:        for (uint i = 0; i < assets.length; i++) {
MarginAccount.sol:552:        for (uint i = 0; i < _collaterals.length; i++) {
MarginAccountHelper.sol:13:    uint constant VUSD_IDX = 0;
```

I suggest removing explicit initializations for default values.

#### Pre-increments cost less gas compared to post-increments

### Comparisons

#### `> 0` is less efficient than `!= 0` for unsigned integers (with proof)

`!= 0` costs less gas compared to `> 0` for unsigned integers in `require` statements with the optimizer enabled (6 gas)

Proof: While it may seem that `> 0` is cheaper than `!=`, this is only true without the optimizer enabled and outside a require statement. If you enable the optimizer at 10k AND you're in a `require` statement, this will save gas. You can see this tweet for more proofs: <https://twitter.com/gzeon/status/1485428085885640706>

`> 0` in require statements are used in the following location(s):

```
AMM.sol:487:        require(baseAssetQuantity > 0, "VAMM._long: baseAssetQuantity is <= 0");
AMM.sol:511:        require(baseAssetQuantity < 0, "VAMM._short: baseAssetQuantity is >= 0");
ClearingHouse.sol:51:        require(_maintenanceMargin > 0, "_maintenanceMargin < 0");
MarginAccount.sol:150:        require(amount > 0, "Add non-zero margin");
Oracle.sol:153:        require(_round > 0, "Not enough history");
```

I suggest you change `> 0` with `!= 0` in require statements. Also, enable the Optimizer.

### For-Loops

#### An array's length should be cached to save gas in for-loops

Reading array length at each iteration of the loop takes 6 gas (3 for mload and 3 to place memory_offset) in the stack.  
  
Caching the array length in the stack saves around 3 gas per iteration.  

Here, I suggest storing the array's length in a variable before the for-loop, and use it instead:

```
ClearingHouse.sol:122:        for (uint i = 0; i < amms.length; i++) {
ClearingHouse.sol:130:        for (uint i = 0; i < amms.length; i++) {
ClearingHouse.sol:170:        for (uint i = 0; i < amms.length; i++) {
ClearingHouse.sol:194:        for (uint i = 0; i < amms.length; i++) { // liquidate all positions
ClearingHouse.sol:251:        for (uint i = 0; i < amms.length; i++) {
ClearingHouse.sol:263:        for (uint i = 0; i < amms.length; i++) {
ClearingHouse.sol:277:        for (uint i = 0; i < amms.length; i++) {
MarginAccount.sol:331:        for (uint i = 0; i < idxs.length; i++) {
MarginAccount.sol:373:        for (uint i = 1 /* skip vusd */; i < assets.length; i++) {
MarginAccount.sol:521:        for (uint i = 0; i < assets.length; i++) {
MarginAccount.sol:552:        for (uint i = 0; i < _collaterals.length; i++) {
```

#### `++i` costs less gas compared to `i++`

`++i` costs less gas compared to `i++` for unsigned integer, as pre-increment is cheaper (about 5 gas per iteration)  

`i++` increments `i` and returns the initial value of `i`. Which means:  
  
```
uint i = 1;  
i++; // == 1 but i == 2  
```
  
But `++i` returns the actual incremented value:  
  
```
uint i = 1;  
++i; // == 2 and i == 2 too, so no need for a temporary variable  
```
  
In the first case, the compiler has to create a temporary variable (when used) for returning `1` instead of `2`  
  
Instances include:  

```
ClearingHouse.sol:122:        for (uint i = 0; i < amms.length; i++) {
ClearingHouse.sol:130:        for (uint i = 0; i < amms.length; i++) {
ClearingHouse.sol:170:        for (uint i = 0; i < amms.length; i++) {
ClearingHouse.sol:194:        for (uint i = 0; i < amms.length; i++) { // liquidate all positions
ClearingHouse.sol:251:        for (uint i = 0; i < amms.length; i++) {
ClearingHouse.sol:263:        for (uint i = 0; i < amms.length; i++) {
ClearingHouse.sol:277:        for (uint i = 0; i < amms.length; i++) {
MarginAccount.sol:331:        for (uint i = 0; i < idxs.length; i++) {
MarginAccount.sol:373:        for (uint i = 1 /* skip vusd */; i < assets.length; i++) {
MarginAccount.sol:521:        for (uint i = 0; i < assets.length; i++) {
MarginAccount.sol:552:        for (uint i = 0; i < _collaterals.length; i++) {
```

I suggest using `++i` instead of `i++` to increment the value of an uint variable.

#### Increments can be unchecked

In Solidity 0.8+, there's a default overflow check on unsigned integers. It's possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck cannot be made inline.  
  
[ethereum/solidity#10695](https://github.com/ethereum/solidity/issues/10695)

Instances include:  

```
ClearingHouse.sol:122:        for (uint i = 0; i < amms.length; i++) {
ClearingHouse.sol:130:        for (uint i = 0; i < amms.length; i++) {
ClearingHouse.sol:170:        for (uint i = 0; i < amms.length; i++) {
ClearingHouse.sol:194:        for (uint i = 0; i < amms.length; i++) { // liquidate all positions
ClearingHouse.sol:251:        for (uint i = 0; i < amms.length; i++) {
ClearingHouse.sol:263:        for (uint i = 0; i < amms.length; i++) {
ClearingHouse.sol:277:        for (uint i = 0; i < amms.length; i++) {
MarginAccount.sol:331:        for (uint i = 0; i < idxs.length; i++) {
MarginAccount.sol:373:        for (uint i = 1 /* skip vusd */; i < assets.length; i++) {
MarginAccount.sol:521:        for (uint i = 0; i < assets.length; i++) {
MarginAccount.sol:552:        for (uint i = 0; i < _collaterals.length; i++) {
```

The code would go from:  
  
```
for (uint256 i; i < numIterations; i++) {  
 // ...  
}  
```

to:  

```
for (uint256 i; i < numIterations;) {  
 // ...  
 unchecked { ++i; }  
}  
```

The risk of overflow is inexistant for a `uint256` here.

### Arithmetics  

#### Shift Right instead of Dividing by 2

A division by 2 can be calculated by shifting one to the right.  
  
While the `DIV` opcode uses 5 gas, the `SHR` opcode only uses 3 gas. Furthermore, Solidity's division operation also includes a division-by-0 prevention which is bypassed using shifting.  

I suggest replacing `/ 2` with `>> 1` here:  

```  
ClearingHouse.sol:212:            uint toInsurance = liquidationFee / 2;
```  

### Errors

#### Reduce the size of error messages (Long revert Strings)

Shortening revert strings to fit in 32 bytes will decrease deployment time gas and will decrease runtime gas when the revert condition is met.

Revert strings that are longer than 32 bytes require at least one additional mstore, along with additional overhead for computing memory offset, etc.

Revert strings > 32 bytes are here:

```
AMM.sol:487:        require(baseAssetQuantity > 0, "VAMM._long: baseAssetQuantity is <= 0");
AMM.sol:511:        require(baseAssetQuantity < 0, "VAMM._short: baseAssetQuantity is >= 0");
ClearingHouse.sol:84:            require(isAboveMinAllowableMargin(trader), "CH: Below Minimum Allowable Margin");
ClearingHouse.sol:101:        require(isAboveMinAllowableMargin(maker), "CH: Below Minimum Allowable Margin");
MarginAccount.sol:174:        require(margin[VUSD_IDX][trader] >= 0, "Cannot remove margin when vusd balance is negative");
MarginAccount.sol:354:        require(notionalPosition == 0, "Liquidate positions before settling bad debt");
MarginAccount.sol:453:        require(repay <= maxRepay, "Need to repay more to seize that much"); 
```

I suggest shortening the revert strings to fit in 32 bytes, or that using custom errors as described next.

#### Use Custom Errors instead of Revert Strings to save Gas

Custom errors from Solidity 0.8.4 are cheaper than revert strings (cheaper deployment cost and runtime cost when the revert condition is met)

Source: <https://blog.soliditylang.org/2021/04/21/custom-errors/>:
> Starting from [Solidity v0.8.4](https://github.com/ethereum/solidity/releases/tag/v0.8.4), there is a convenient and gas-efficient way to explain to users why an operation failed through the use of custom errors. Until now, you could already use strings to give more information about failures (e.g., `revert("Insufficient funds.");`), but they are rather expensive, especially when it comes to deploy cost, and it is difficult to use dynamic information in them.

Custom errors are defined using the `error` statement, which can be used inside and outside of contracts (including interfaces and libraries).

Instances include:

```
legos/Governable.sol:11:        require(msg.sender == governance, "ONLY_GOVERNANCE");
AMM.sol:84:        require(msg.sender == clearingHouse, "Only clearingHouse");
AMM.sol:89:        require(msg.sender == address(vamm), "Only VAMM");
AMM.sol:119:        require(ammState == AMMState.Active, "AMM.openPosition.not_active");
AMM.sol:186:        require(ammState != AMMState.Inactive, "AMM.addLiquidity.amm_inactive");
AMM.sol:296:        require(abs(positionSize) >= abs(baseAssetQuantity), "AMM.ONLY_REDUCE_POS");
AMM.sol:348:        require(_blockTimestamp() >= nextFundingTime, "settle funding too early");
AMM.sol:487:        require(baseAssetQuantity > 0, "VAMM._long: baseAssetQuantity is <= 0");
AMM.sol:511:        require(baseAssetQuantity < 0, "VAMM._short: baseAssetQuantity is >= 0");
AMM.sol:723:        require(ammState != _state, "AMM.setAmmState.sameState");
ClearingHouse.sol:51:        require(_maintenanceMargin > 0, "_maintenanceMargin < 0");
ClearingHouse.sol:75:        require(baseAssetQuantity != 0, "CH: baseAssetQuantity == 0");
ClearingHouse.sol:84:            require(isAboveMinAllowableMargin(trader), "CH: Below Minimum Allowable Margin");
ClearingHouse.sol:101:        require(isAboveMinAllowableMargin(maker), "CH: Below Minimum Allowable Margin");
ClearingHouse.sol:120:        require(address(trader) != address(0), 'CH: 0x0 trader Address');
ClearingHouse.sol:154:        require(!isMaker(trader), 'CH: Remove Liquidity First');
ClearingHouse.sol:164:        require(
ClearingHouse.sol:189:        require(_calcMarginFraction(trader, false /* check funding payments again */) < maintenanceMargin, "Above Maintenance Margin");
InsuranceFund.sol:30:        require(msg.sender == address(marginAccount), "IF.only_margin_account");
InsuranceFund.sol:42:        require(pendingObligation == 0, "IF.deposit.pending_obligations");
InsuranceFund.sol:64:        require(pendingObligation == 0, "IF.withdraw.pending_obligations");
MarginAccount.sol:115:        require(_msgSender() == address(clearingHouse), "Only clearingHouse");
MarginAccount.sol:150:        require(amount > 0, "Add non-zero margin");
MarginAccount.sol:174:        require(margin[VUSD_IDX][trader] >= 0, "Cannot remove margin when vusd balance is negative");
MarginAccount.sol:175:        require(margin[idx][trader] >= amount.toInt256(), "Insufficient balance");
MarginAccount.sol:180:        require(clearingHouse.isAboveMinAllowableMargin(trader), "MA.removeMargin.Below_MM");
MarginAccount.sol:354:        require(notionalPosition == 0, "Liquidate positions before settling bad debt");
MarginAccount.sol:357:        require(getSpotCollateralValue(trader) < 0, "Above bad debt threshold");
MarginAccount.sol:362:        require(vusdBal < 0, "Nothing to repay");
MarginAccount.sol:438:        require(seized >= minSeizeAmount, "Not seizing enough");
MarginAccount.sol:453:        require(repay <= maxRepay, "Need to repay more to seize that much");
MarginAccount.sol:461:        require(idx > VUSD_IDX && idx < supportedCollateral.length, "collateral not seizable");
MarginAccount.sol:549:        require(_weight <= PRECISION, "weight > 1e6");
MarginAccount.sol:553:            require(address(_collaterals[i].token) != _coin, "collateral exists");
MarginAccount.sol:601:        require(_liquidationIncentive <= PRECISION / 10, "MA.syncDeps.LI_GT_10_percent");
MarginAccount.sol:603:        require(registry.marginAccount() == address(this), "Incorrect setup");
MarginAccount.sol:617:        require(_weight <= PRECISION, "weight > 1e6");
MarginAccount.sol:618:        require(idx < supportedCollateral.length, "Collateral not supported");
MinimalForwarder.sol:15:        require(success, string(abi.encodePacked("META_EXEC_FAILED: ", returnData)));
Oracle.sol:48:        require(intervalInSeconds != 0, "interval can't be 0");
Oracle.sol:153:        require(_round > 0, "Not enough history");
Oracle.sol:157:        require(_addr != address(0), "empty address");
VUSD.sol:33:        require(_reserveToken != address(0), "vUSD: null _reserveToken");
VUSD.sol:55:        require(reserve >= withdrawals[start].amount, 'Cannot process withdrawals at this time: Not enough balance');
```

I suggest replacing revert strings with custom errors.
