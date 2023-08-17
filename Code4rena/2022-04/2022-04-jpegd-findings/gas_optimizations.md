## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas Optimizations](https://github.com/code-423n4/2022-04-jpegd-findings/issues/121) 

**Table of Contents:**

- [`NFTEscrow._executeTransfer()`: Cheap Contract Deployment Through Clones](#nftescrow_executetransfer-cheap-contract-deployment-through-clones)
- [`LPFarming.newEpoch()`: L128 and L133 should be unchecked due to parent if/else condition](#lpfarmingnewepoch-l128-and-l133-should-be-unchecked-due-to-parent-ifelse-condition)
- [`LPFarming.withdraw()`: L248 should be unchecked due to L243](#lpfarmingwithdraw-l248-should-be-unchecked-due-to-l243)
- [`LPFarming._withdrawReward()`: `poolInfo[_pid].accRewardPerShare` should get cached](#lpfarming_withdrawreward-poolinfo_pidaccrewardpershare-should-get-cached)
- [`yVaultLPFarming.withdraw()`: L124 should be unchecked due to L119](#yvaultlpfarmingwithdraw-l124-should-be-unchecked-due-to-l119)
- [`yVaultLPFarming._withdrawReward()`: `accRewardPerShare` should get cached](#yvaultlpfarming_withdrawreward-accrewardpershare-should-get-cached)
- [`JPEGLock.unlock()`: use `storage` instead of copying struct in memory L69](#jpeglockunlock-use-storage-instead-of-copying-struct-in-memory-l69)
- [`FungibleAssetVaultForDAO._collateralPriceUsd()`: `oracle` should get cached](#fungibleassetvaultfordao_collateralpriceusd-oracle-should-get-cached)
- [`FungibleAssetVaultForDAO._collateralPriceUsd()`: return statement should be unchecked](#fungibleassetvaultfordao_collateralpriceusd-return-statement-should-be-unchecked)
- [`FungibleAssetVaultForDAO.deposit()`: `collateralAsset` should get cached](#fungibleassetvaultfordaodeposit-collateralasset-should-get-cached)
- [`FungibleAssetVaultForDAO.repay)`: L184 should be unchecked due to L182](#fungibleassetvaultfordaorepay-l184-should-be-unchecked-due-to-l182)
- [`FungibleAssetVaultForDAO.withdraw()`: L196 should be unchecked due to L194](#fungibleassetvaultfordaowithdraw-l196-should-be-unchecked-due-to-l194)
- [`FungibleAssetVaultForDAO.withdraw()`: `collateralAmount` should get cached](#fungibleassetvaultfordaowithdraw-collateralamount-should-get-cached)
- [`NFTVault._normalizeAggregatorAnswer()`: return statement should be unchecked](#nftvault_normalizeaggregatoranswer-return-statement-should-be-unchecked)
- [`NFTVault._calculateAdditionalInterest()`: `totalDebtAmount` should get cached](#nftvault_calculateadditionalinterest-totaldebtamount-should-get-cached)
- [`NFTVault.sol`: `struct PositionPreview` can be tightly packed to save 1 storage slot](#nftvaultsol-struct-positionpreview-can-be-tightly-packed-to-save-1-storage-slot)
- [`NFTVault.showPosition()`: L659 should be unchecked due to L649](#nftvaultshowposition-l659-should-be-unchecked-due-to-l649)
- [`NFTVault.showPosition()`: `positions[_nftIndex].liquidatedAt` should get cached](#nftvaultshowposition-positions_nftindexliquidatedat-should-get-cached)
- [`NFTVault.showPosition()`: Help the optimizer by saving a storage variable's reference instead of repeatedly fetching it (`positions[_nftIndex]`)](#nftvaultshowposition-help-the-optimizer-by-saving-a-storage-variables-reference-instead-of-repeatedly-fetching-it-positions_nftindex)
- [`NFTVault.borrow()`: `totalDebtPortion` should get cached](#nftvaultborrow-totaldebtportion-should-get-cached)
- [`NFTVault.repay()`: L781 should be unchecked due to ternary operator](#nftvaultrepay-l781-should-be-unchecked-due-to-ternary-operator)
- [`NFTVault.repay()`: `totalDebtPortion` and `totalDebtAmount` should get cached](#nftvaultrepay-totaldebtportion-and-totaldebtamount-should-get-cached)
- [`Controller.setStrategy()`: boolean comparison L87](#controllersetstrategy-boolean-comparison-l87)
- [`StrategyPUSDConvex.balanceOfJPEG()`: `jpeg` should get cached](#strategypusdconvexbalanceofjpeg-jpeg-should-get-cached)
- [`StrategyPUSDConvex.balanceOfJPEG()`: use a `return` statement instead of `break`](#strategypusdconvexbalanceofjpeg-use-a-return-statement-instead-of-break)
- [`StrategyPUSDConvex.withdraw()`: L281 should be unchecked due to L279](#strategypusdconvexwithdraw-l281-should-be-unchecked-due-to-l279)
- [`StrategyPUSDConvex.harvest()`: L362 should be unchecked due to L359-L360 and how performanceFee is set L183](#strategypusdconvexharvest-l362-should-be-unchecked-due-to-l359-l360-and-how-performancefee-is-set-l183)
- [`yVault.earn()`: `token` and `controller` should get cached](#yvaultearn-token-and-controller-should-get-cached)
- [`yVault.withdraw()`: L178 should be unchecked due to L177](#yvaultwithdraw-l178-should-be-unchecked-due-to-l177)
- [`yVault.withdraw()`: `token` should get cached](#yvaultwithdraw-token-should-get-cached)
- [`yVault.withdrawJPEG()`: `farm` should get cached](#yvaultwithdrawjpeg-farm-should-get-cached)
- [Upgrade pragma to at least 0.8.4](#upgrade-pragma-to-at-least-084)
- [No need to explicitly initialize variables with default values](#no-need-to-explicitly-initialize-variables-with-default-values)
- [`> 0` is less efficient than `!= 0` for unsigned integers (with proof)](#-0-is-less-efficient-than--0-for-unsigned-integers-with-proof)
- [`>=` is cheaper than `>`](#-is-cheaper-than-)
- [An array's length should be cached to save gas in for-loops](#an-arrays-length-should-be-cached-to-save-gas-in-for-loops)
- [`++i` costs less gas compared to `i++` or `i += 1`](#i-costs-less-gas-compared-to-i-or-i--1)
- [Increments can be unchecked](#increments-can-be-unchecked)
- [Use `calldata` instead of `memory`](#use-calldata-instead-of-memory)
- [Consider making some constants as non-public to save gas](#consider-making-some-constants-as-non-public-to-save-gas)
- [Public functions to external](#public-functions-to-external)
- [Reduce the size of error messages (Long revert Strings)](#reduce-the-size-of-error-messages-long-revert-strings)
- [Use Custom Errors instead of Revert Strings to save Gas](#use-custom-errors-instead-of-revert-strings-to-save-gas)

## `NFTEscrow._executeTransfer()`: Cheap Contract Deployment Through Clones

See `@audit` tag:

```solidity
67:     function _executeTransfer(address _owner, uint256 _idx) internal {
68:         (bytes32 salt, ) = precompute(_owner, _idx);
69:         new FlashEscrow{salt: salt}( //@audit gas: deployment can cost less through clones 
70:             nftAddress,
71:             _encodeFlashEscrowPayload(_idx)
72:         );
73:     }
```

There's a way to save a significant amount of gas on deployment using Clones: <https://www.youtube.com/watch?v=3Mw-pMmJ7TA> .

This is a solution that was adopted, as an example, by Porter Finance. They realized that deploying using clones was 10x cheaper:

- <https://github.com/porter-finance/v1-core/issues/15#issuecomment-1035639516>
- <https://github.com/porter-finance/v1-core/pull/34>

I suggest applying a similar pattern.

## `LPFarming.newEpoch()`: L128 and L133 should be unchecked due to parent if/else condition

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn't possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an `unchecked` block: <https://docs.soliditylang.org/en/v0.8.10/control-structures.html#checked-or-unchecked-arithmetic>

I suggest wrapping with an `unchecked` block here (see `@audit` tag):

```solidity
107:     function newEpoch(
...
111:     ) external onlyOwner {
127:         if (remainingRewards > newRewards) {
128:             jpeg.safeTransfer(msg.sender, remainingRewards - newRewards);  //@audit gas: should be unchecked (can't underflow due to L127)
129:         } else if (remainingRewards < newRewards) {
130:             jpeg.safeTransferFrom(
131:                 msg.sender,
132:                 address(this),
133:                 newRewards - remainingRewards  //@audit gas: should be unchecked (can't underflow due to L129)
134:             );
135:         }
136:     }
```

## `LPFarming.withdraw()`: L248 should be unchecked due to L243

See `@audit` tag:

```solidity
235:     function withdraw(uint256 _pid, uint256 _amount)
236:         external
237:         noContract(msg.sender)
238:     {
239:         require(_amount > 0, "invalid_amount");
240: 
241:         PoolInfo storage pool = poolInfo[_pid];
242:         UserInfo storage user = userInfo[_pid][msg.sender];
243:         require(user.amount >= _amount, "insufficient_amount");
244: 
245:         _updatePool(_pid);
246:         _withdrawReward(_pid);
247: 
248:         user.amount -= _amount;  //@audit gas: should be unchecked (can't underflow due to L243)
```

## `LPFarming._withdrawReward()`: `poolInfo[_pid].accRewardPerShare` should get cached

The code can be optimized by minimising the number of SLOADs. SLOADs are expensive (100 gas) compared to MLOADs/MSTOREs (3 gas). Here, the storage value should get cached in memory (see the `@audit` tags for further details):

```solidity
315:     function _withdrawReward(uint256 _pid) internal returns (uint256) {
316:         UserInfo storage user = userInfo[_pid][msg.sender];
317:         uint256 pending = (user.amount *
318:             (poolInfo[_pid].accRewardPerShare - user.lastAccRewardPerShare)) / //@audit gas: SLOAD 1 poolInfo[_pid].accRewardPerShare
319:             1e36;
320:         if (pending > 0) {
321:             userRewards[msg.sender] += pending;
322:         }
323: 
324:         user.lastAccRewardPerShare = poolInfo[_pid].accRewardPerShare; //@audit gas: SLOAD 2 poolInfo[_pid].accRewardPerShare
325: 
326:         return pending;
327:     }
```

## `yVaultLPFarming.withdraw()`: L124 should be unchecked due to L119

See `@audit` tag:

```solidity
117:     function withdraw(uint256 _amount) external noContract(msg.sender) {
118:         require(_amount > 0, "invalid_amount");
119:         require(balanceOf[msg.sender] >= _amount, "insufficient_amount");
120: 
121:         _update();
122:         _withdrawReward(msg.sender);
123: 
124:         balanceOf[msg.sender] -= _amount;  //@audit gas: should be unchecked (can't underflow due to L119)
```

## `yVaultLPFarming._withdrawReward()`: `accRewardPerShare` should get cached

See `@audit` tags:

```solidity
177:     function _withdrawReward(address account) internal returns (uint256) {
178:         uint256 pending = (balanceOf[account] *
179:             (accRewardPerShare - userLastAccRewardPerShare[account])) / 1e36; //@audit gas: SLOAD 1 accRewardPerShare
180: 
181:         if (pending > 0) userPendingRewards[account] += pending;
182: 
183:         userLastAccRewardPerShare[account] = accRewardPerShare; //@audit gas: SLOAD 2 accRewardPerShare
184: 
185:         return pending;
186:     }
```

## `JPEGLock.unlock()`: use `storage` instead of copying struct in memory L69

See `@audit` tag:

```solidity
68:     function unlock(uint256 _nftIndex) external nonReentrant {
69:         LockPosition memory position = positions[_nftIndex]; //@audit gas: costing 3 SLOADs while only lockAmount is needed twice. Replace "memory" with "storage" and cache only position.lockAmount 
70:         require(position.owner == msg.sender, "unauthorized");
71:         require(position.unlockAt <= block.timestamp, "locked");
72: 
73:         delete positions[_nftIndex];
74: 
75:         jpeg.safeTransfer(msg.sender, position.lockAmount);
76: 
77:         emit Unlock(msg.sender, _nftIndex, position.lockAmount);
78:     }
```

Here, a copy in memory is costing 3 SLOADs and 3 MSTORES. The, 2 variables are only read once through MLOAD (`position.owner` and `position.unlockAt`) and one is read twice (`position.lockAmount`).
I suggest replacing the `memory` keyword with `storage` at L69 and only copying `position.lockAmount` in memory.

## `FungibleAssetVaultForDAO._collateralPriceUsd()`: `oracle` should get cached

See `@audit` tags:

```solidity
104:     function _collateralPriceUsd() internal view returns (uint256) {
105:         int256 answer = oracle.latestAnswer();  //@audit gas: SLOAD 1 oracle
106:         uint8 decimals = oracle.decimals();  //@audit gas: SLOAD 2 oracle
107: 
```

## `FungibleAssetVaultForDAO._collateralPriceUsd()`: return statement should be unchecked

See `@audit` tag:

```solidity
104:     function _collateralPriceUsd() internal view returns (uint256) {
...
111:         return //@audit gas: whole return statement should be unchecked (obviously can't underflow/overflow here)
112:             decimals > 18
113:                 ? uint256(answer) / 10**(decimals - 18)  
114:                 : uint256(answer) * 10**(18 - decimals);  
115:     }
```

Due to the ternary condition and the fact that `int256 answer = oracle.latestAnswer();`, the return statement can't underflow and should be unchecked.

## `FungibleAssetVaultForDAO.deposit()`: `collateralAsset` should get cached

See `@audit` tags:

```solidity
141:     function deposit(uint256 amount) external payable onlyRole(WHITELISTED_ROLE) {
142:         require(amount > 0, "invalid_amount");
143: 
144:         if (collateralAsset == ETH) {  //@audit gas: SLOAD 1 collateralAsset
145:             require(msg.value == amount, "invalid_msg_value");
146:         } else {
147:             require(msg.value == 0, "non_zero_eth_value");
148:             IERC20Upgradeable(collateralAsset).safeTransferFrom(  //@audit gas: SLOAD 2 collateralAsset
149:                 msg.sender,
150:                 address(this),
151:                 amount
152:             );
153:         }
```

## `FungibleAssetVaultForDAO.repay)`: L184 should be unchecked due to L182

See `@audit` tag:

```solidity
179:     function repay(uint256 amount) external onlyRole(WHITELISTED_ROLE) nonReentrant {
180:         require(amount > 0, "invalid_amount");
181: 
182:         amount = amount > debtAmount ? debtAmount : amount;
183: 
184:         debtAmount -= amount; //@audit gas: should be unchecked (can't underflow due to L182)
```

## `FungibleAssetVaultForDAO.withdraw()`: L196 should be unchecked due to L194

See `@audit` tag:

```solidity
193:     function withdraw(uint256 amount) external onlyRole(WHITELISTED_ROLE) nonReentrant {
194:         require(amount > 0 && amount <= collateralAmount, "invalid_amount");
195: 
196:         uint256 creditLimit = getCreditLimit(collateralAmount - amount); //@audit gas: should be unchecked (can't underflow due to L194)
```

## `FungibleAssetVaultForDAO.withdraw()`: `collateralAmount` should get cached

See `@audit` tags:

```solidity
193:     function withdraw(uint256 amount) external onlyRole(WHITELISTED_ROLE) nonReentrant {
194:         require(amount > 0 && amount <= collateralAmount, "invalid_amount");  //@audit gas: SLOAD 1 collateralAmount
195: 
196:         uint256 creditLimit = getCreditLimit(collateralAmount - amount); //@audit gas: SLOAD 2 collateralAmount
197:         require(creditLimit >= debtAmount, "insufficient_credit");
198: 
199:         collateralAmount -= amount; //@audit gas: SLOAD 3 collateralAmount (could've used the a cached value for calculation)
```

## `NFTVault._normalizeAggregatorAnswer()`: return statement should be unchecked

See `@audit` tag:

```solidity
454:     function _normalizeAggregatorAnswer(IAggregatorV3Interface aggregator)
455:         internal
456:         view
457:         returns (uint256)
458:     {
...
464:         return //@audit gas: whole return statement should be unchecked (obviously can't underflow/overflow)
465:             decimals > 18
466:                 ? uint256(answer) / 10**(decimals - 18)
467:                 : uint256(answer) * 10**(18 - decimals);
468:     }
```

## `NFTVault._calculateAdditionalInterest()`: `totalDebtAmount` should get cached

See `@audit` tags:

```solidity
578:     function _calculateAdditionalInterest() internal view returns (uint256) {
...
585:         if (totalDebtAmount == 0) {  //@audit gas: SLOAD 1 totalDebtAmount
586:             return 0;
587:         }
588: 
589:         // Accrue interest
590:         uint256 interestPerYear = (totalDebtAmount *  //@audit gas: SLOAD 2 totalDebtAmount
```

## `NFTVault.sol`: `struct PositionPreview` can be tightly packed to save 1 storage slot

From (see `@audit` tags):

```solidity
610:     struct PositionPreview { // @audit gas: can be tightly packed by moving borrowType and liquidatable at the end
611:         address owner;
612:         uint256 nftIndex;
613:         bytes32 nftType;
614:         uint256 nftValueUSD;
615:         VaultSettings vaultSettings;
616:         uint256 creditLimit;
617:         uint256 debtPrincipal;
618:         uint256 debtInterest; // @audit gas: 32 bytes
619:         BorrowType borrowType; // @audit gas: 1 byte (this enum is equivalent to uint8 as it has less than 256 options)
620:         bool liquidatable; // @audit gas: 1 byte
621:         uint256 liquidatedAt; // @audit gas: 32 bytes
622:         address liquidator; // @audit gas: 20 bytes
623:     }
```

To:

```solidity
    struct PositionPreview {
        address owner;
        uint256 nftIndex;
        bytes32 nftType;
        uint256 nftValueUSD;
        VaultSettings vaultSettings;
        uint256 creditLimit;
        uint256 debtPrincipal;
        uint256 debtInterest; // @audit gas: 32 bytes
        uint256 liquidatedAt; // @audit gas: 32 bytes
        BorrowType borrowType; // @audit gas: 1 byte (this enum is equivalent to uint8 as it has less than 256 options)
        bool liquidatable; // @audit gas: 1 byte
        address liquidator; // @audit gas: 20 bytes
    }
```

## `NFTVault.showPosition()`: L659 should be unchecked due to L649

See `@audit` tag:

```solidity
File: NFTVault.sol
628:     function showPosition(uint256 _nftIndex)
...
649:         if (debtPrincipal > debtAmount) debtAmount = debtPrincipal;
...
659:             debtInterest: debtAmount - debtPrincipal, //@audit gas: should be unchecked (can't underflow due to L649)
```

## `NFTVault.showPosition()`: `positions[_nftIndex].liquidatedAt` should get cached

See `@audit` tags:

```solidity
File: NFTVault.sol
628:     function showPosition(uint256 _nftIndex)
...
661:             liquidatable: positions[_nftIndex].liquidatedAt == 0 &&  //@audit gas: SLOAD 1 positions[_nftIndex].liquidatedAt
662:                 debtAmount >= _getLiquidationLimit(_nftIndex),
663:             liquidatedAt: positions[_nftIndex].liquidatedAt,  //@audit gas: SLOAD 2 positions[_nftIndex].liquidatedAt
```

## `NFTVault.showPosition()`: Help the optimizer by saving a storage variable's reference instead of repeatedly fetching it (`positions[_nftIndex]`)

To help the optimizer, declare a `storage` type variable and use it instead of repeatedly fetching the reference in a map or an array.

The effect can be quite significant.

Here, instead of repeatedly calling `positions[_nftIndex]`, save its reference like this: `Position storage _position = positions[_nftIndex]` and use it.

Impacted lines (see `@audit` tags):

```solidity
  636:         uint256 debtPrincipal = positions[_nftIndex].debtPrincipal; //@audit gas: use the suggested storage variable "Position storage _position"
  637:         uint256 debtAmount = positions[_nftIndex].liquidatedAt > 0 //@audit gas: use the suggested storage variable "Position storage _position"
  638:             ? positions[_nftIndex].debtAmountForRepurchase //@audit gas: use the suggested storage variable "Position storage _position"
  641:                 positions[_nftIndex].debtPortion, //@audit gas: use the suggested storage variable "Position storage _position"
  660:             borrowType: positions[_nftIndex].borrowType, //@audit gas: use the suggested storage variable "Position storage _position"
  661:             liquidatable: positions[_nftIndex].liquidatedAt == 0 && //@audit gas: use the suggested storage variable "Position storage _position"
  663:             liquidatedAt: positions[_nftIndex].liquidatedAt, //@audit gas: use the suggested storage variable "Position storage _position"
  664:             liquidator: positions[_nftIndex].liquidator //@audit gas: use the suggested storage variable "Position storage _position"
```

This practice already exists in the solution, as seen in `NFTVault.borrow()`:

```solidity
675:     function borrow(
...
697:         Position storage position = positions[_nftIndex];
```

## `NFTVault.borrow()`: `totalDebtPortion` should get cached

See `@audit` tags:

```solidity
675:     function borrow(
...
735:         if (totalDebtPortion == 0) {  //@audit gas: SLOAD 1 totalDebtPortion
...
738:         } else {
739:             uint256 plusPortion = (totalDebtPortion * _amount) / //@audit gas: SLOAD 2 totalDebtPortion
740:                 totalDebtAmount;
741:             totalDebtPortion += plusPortion; //@audit gas: SLOAD 3 totalDebtPortion (could've used cached value for calculation)
```

## `NFTVault.repay()`: L781 should be unchecked due to ternary operator

See `@audit` tag:

```solidity
756:     function repay(uint256 _nftIndex, uint256 _amount)
...
780:         uint256 paidPrincipal = _amount > debtInterest
781:             ? _amount - debtInterest //@audit gas: should be unchecked (obviously)
782:             : 0;
```

## `NFTVault.repay()`: `totalDebtPortion` and `totalDebtAmount` should get cached

See `@audit` tags:

```solidity
756:     function repay(uint256 _nftIndex, uint256 _amount)
...
784:         uint256 minusPortion = paidPrincipal == debtPrincipal
785:             ? position.debtPortion
786:             : (totalDebtPortion * _amount) / totalDebtAmount; //@audit gas: SLOADs 1 totalDebtPortion & totalDebtAmount
787: 
788:         totalDebtPortion -= minusPortion; //@audit gas: SLOAD 2 totalDebtPortion (could've used cached value for calculation)
...
791:         totalDebtAmount -= _amount; //@audit gas: SLOAD 2 totalDebtAmount (could've used cached value for calculation)
```

## `Controller.setStrategy()`: boolean comparison L87

Comparing to a constant (`true` or `false`) is a bit more expensive than directly checking the returned boolean value.
I suggest using `if(directValue)` instead of `if(directValue == true)` and `if(!directValue)` instead of `if(directValue == false)` here (see `@audit` tag):

```solidity
82:     function setStrategy(IERC20 _token, IStrategy _strategy)
83:         external
84:         onlyRole(STRATEGIST_ROLE)
85:     {
86:         require(
87:             approvedStrategies[_token][_strategy] == true, //@audit gas: instead of comparing to a constant, just use "approvedStrategies[_token][_strategy]"
```

## `StrategyPUSDConvex.balanceOfJPEG()`: `jpeg` should get cached

See `@audit` tags:

```solidity
226:     function balanceOfJPEG() external view returns (uint256) {
227:         uint256 availableBalance = jpeg.balanceOf(address(this)); //@audit gas: SLOAD 1 jpeg
...
231:         for (uint256 i = 0; i < length; i++) {
...
233:             if (address(jpeg) == extraReward.rewardToken()) { //@audit gas: SLOADs in Loop for jpeg. Cache it at L227
```

## `StrategyPUSDConvex.balanceOfJPEG()`: use a `return` statement instead of `break`

See `@audit` tag:

```solidity
226:     function balanceOfJPEG() external view returns (uint256) {
...
231:         for (uint256 i = 0; i < length; i++) {
...
233:             if (address(jpeg) == extraReward.rewardToken()) {
234:                 availableBalance += extraReward.earned();
235:                 //we found jpeg, no need to continue the loop
236:                 break; //@audit gas: instead of adding to availableBalance & breaking, just return here "availableBalance + extraReward.earned()"
237:             }
238:         }
239: 
240:         return availableBalance;
241:     }
```

Here, instead of making a memory operation with `availableBalance += extraReward.earned();` and then using `break;` before returning the memory variable `availableBalance`, it would've been more optimized to just return `availableBalance + extraReward.earned()`:

```solidity
    function balanceOfJPEG() external view returns (uint256) {
...
        for (uint256 i = 0; i < length; i++) {
...
            if (address(jpeg) == extraReward.rewardToken()) {
              return availableBalance + extraReward.earned();
            }
        }
    }
```

## `StrategyPUSDConvex.withdraw()`: L281 should be unchecked due to L279

See `@audit` tag:

```solidity
273:     function withdraw(uint256 _amount) external onlyController {
...
279:         if (balance < _amount)
280:             convexConfig.baseRewardPool.withdrawAndUnwrap(
281:                 _amount - balance, //@audit gas: should be unchecked (can't underflow due to L279)
282:                 false
283:             );
```

## `StrategyPUSDConvex.harvest()`: L362 should be unchecked due to L359-L360 and how performanceFee is set L183

See `@audit` tags:

```solidity
177:     function setPerformanceFee(Rate memory _performanceFee)
...
181:         require(
182:             _performanceFee.denominator > 0 &&
183:                 _performanceFee.denominator >= _performanceFee.numerator, //@audit gas: can uncheck L362 thanks to this
184:             "INVALID_RATE"
185:         );
...
311:     function harvest(uint256 minOutCurve) external onlyRole(STRATEGIST_ROLE) {
...
359:         uint256 fee = (usdcBalance * performanceFee.numerator) /
360:             performanceFee.denominator;
361:         usdc.safeTransfer(strategy.controller.feeAddress(), fee);
362:         usdcBalance -= fee; //@audit gas: should be unchecked (can't underflow due to L359-L360 & how performanceFee is set L183)
```

## `yVault.earn()`: `token` and `controller` should get cached

See `@audit` tags:

```solidity
129:     function earn() external {
130:         uint256 _bal = available();
131:         token.safeTransfer(address(controller), _bal); //@audit gas: SLOADs 1 token & controller
132:         controller.earn(address(token), _bal); //@audit gas: SLOADs 2 token & controller
133:     }
```

## `yVault.withdraw()`: L178 should be unchecked due to L177

See `@audit` tag:

```solidity
166:     function withdraw(uint256 _shares) public noContract(msg.sender) {
...
177:         if (vaultBalance < backingTokens) {
178:             uint256 toWithdraw = backingTokens - vaultBalance; //@audit gas: should be unchecked (can't underflow due to L177)
```

## `yVault.withdraw()`: `token` should get cached

See `@audit` tags:

```solidity
166:     function withdraw(uint256 _shares) public noContract(msg.sender) {
...
176:         uint256 vaultBalance = token.balanceOf(address(this)); //@audit gas: SLOAD 1 token
177:         if (vaultBalance < backingTokens) {
178:             uint256 toWithdraw = backingTokens - vaultBalance;
179:             controller.withdraw(address(token), toWithdraw);  //@audit gas: SLOAD 2 token
180:         }
181: 
182:         token.safeTransfer(msg.sender, backingTokens);  //@audit gas: SLOAD 3 token
```

## `yVault.withdrawJPEG()`: `farm` should get cached

See `@audit` tags:

```solidity
187:     function withdrawJPEG() external {
188:         require(farm != address(0), "NO_FARM");  //@audit gas: SLOAD 1 farm
189:         controller.withdrawJPEG(address(token), farm);  //@audit gas: SLOAD 2 farm
190:     }
```

## Upgrade pragma to at least 0.8.4

Across the whole solution, the declared pragma is `^0.8.0`.

Using newer compiler versions and the optimizer give gas optimizations. Also, additional safety checks are available for free.

The advantages here are:

- **Low level inliner** (>= 0.8.2): Cheaper runtime gas (especially relevant when the contract has small functions).
- **Optimizer improvements in packed structs** (>= 0.8.3)
- **Custom errors** (>= 0.8.4): cheaper deployment cost and runtime cost. *Note*: the runtime cost is only relevant when the revert condition is met. In short, replace revert strings by custom errors.

Consider upgrading pragma to at least 0.8.4:

## No need to explicitly initialize variables with default values

If a variable is not set/initialized, it is assumed to have the default value (`0` for `uint`, `false` for `bool`, `address(0)` for address...). Explicitly initializing it with its default value is an anti-pattern and wastes gas.

As an example: `for (uint256 i = 0; i < numIterations; ++i) {` should be replaced with `for (uint256 i; i < numIterations; ++i) {`

Instances include:  

```solidity
farming/LPFarming.sol:281:        for (uint256 pid = 0; pid < length; ++pid) {
farming/LPFarming.sol:348:        for (uint256 i = 0; i < poolInfo.length; i++) {
vaults/yVault/strategies/StrategyPUSDConvex.sol:145:        for (uint256 i = 0; i < _strategyConfig.rewardTokens.length; i++) {
vaults/yVault/strategies/StrategyPUSDConvex.sol:231:        for (uint256 i = 0; i < length; i++) {
vaults/yVault/strategies/StrategyPUSDConvex.sol:319:            for (uint256 i = 0; i < rewardTokens.length; i++) {
vaults/FungibleAssetVaultForDAO.sol:45:    address internal constant ETH = address(0);
vaults/NFTVault.sol:181:        for (uint256 i = 0; i < _typeInitializers.length; i++) {
vaults/NFTVault.sol:184:            for (uint256 j = 0; j < initializer.nfts.length; j++) {
```

I suggest removing explicit initializations for default values.

## `> 0` is less efficient than `!= 0` for unsigned integers (with proof)

`!= 0` costs less gas compared to `> 0` for unsigned integers in `require` statements with the optimizer enabled (6 gas)

Proof: While it may seem that `> 0` is cheaper than `!=`, this is only true without the optimizer enabled and outside a require statement. If you enable the optimizer at 10k AND you're in a `require` statement, this will save gas. You can see this tweet for more proofs: <https://twitter.com/gzeon/status/1485428085885640706>

I suggest changing `> 0` with `!= 0` here:

```solidity
farming/LPFarming.sol:114:        require(_rewardPerBlock > 0, "Invalid reward per block");
farming/LPFarming.sol:218:        require(_amount > 0, "invalid_amount");
farming/LPFarming.sol:239:        require(_amount > 0, "invalid_amount");
farming/LPFarming.sol:337:        require(rewards > 0, "no_reward");
farming/LPFarming.sol:354:        require(rewards > 0, "no_reward");
farming/yVaultLPFarming.sol:101:        require(_amount > 0, "invalid_amount");
farming/yVaultLPFarming.sol:118:        require(_amount > 0, "invalid_amount");
farming/yVaultLPFarming.sol:139:        require(rewards > 0, "no_reward");
lock/JPEGLock.sol:40:        require(_newTime > 0, "Invalid lock time");
staking/JPEGStaking.sol:32:        require(_amount > 0, "invalid_amount");
vaults/yVault/strategies/StrategyPUSDConvex.sol:182:            _performanceFee.denominator > 0 &&
vaults/yVault/strategies/StrategyPUSDConvex.sol:334:            require(wethBalance > 0, "NOOP");
vaults/yVault/yVault.sol:100:            _rate.numerator > 0 && _rate.denominator >= _rate.numerator,
vaults/yVault/yVault.sol:143:        require(_amount > 0, "INVALID_AMOUNT");
vaults/yVault/yVault.sol:167:        require(_shares > 0, "INVALID_AMOUNT");
vaults/yVault/yVault.sol:170:        require(supply > 0, "NO_TOKENS_DEPOSITED");
vaults/FungibleAssetVaultForDAO.sol:94:            _creditLimitRate.denominator > 0 &&
vaults/FungibleAssetVaultForDAO.sol:108:        require(answer > 0, "invalid_oracle_answer");
vaults/FungibleAssetVaultForDAO.sol:142:        require(amount > 0, "invalid_amount");
vaults/FungibleAssetVaultForDAO.sol:164:        require(amount > 0, "invalid_amount");
vaults/FungibleAssetVaultForDAO.sol:180:        require(amount > 0, "invalid_amount");
vaults/FungibleAssetVaultForDAO.sol:194:        require(amount > 0 && amount <= collateralAmount, "invalid_amount");
vaults/NFTVault.sol:278:        require(_newFloor > 0, "Invalid floor");
vaults/NFTVault.sol:327:            _type == bytes32(0) || nftTypeValueETH[_type] > 0,
vaults/NFTVault.sol:365:        require(pendingValue > 0, "no_pending_value");
vaults/NFTVault.sol:402:            rate.denominator > 0 && rate.denominator >= rate.numerator,
vaults/NFTVault.sol:462:        require(answer > 0, "invalid_oracle_answer");
vaults/NFTVault.sol:687:        require(_amount > 0, "invalid_amount");
vaults/NFTVault.sol:764:        require(_amount > 0, "invalid_amount");
vaults/NFTVault.sol:770:        require(debtAmount > 0, "position_not_borrowed");
vaults/NFTVault.sol:882:        require(position.liquidatedAt > 0, "not_liquidated");
vaults/NFTVault.sol:926:        require(position.liquidatedAt > 0, "not_liquidated");
```

Also, please enable the Optimizer.

## `>=` is cheaper than `>`

Strict inequalities (`>`) are more expensive than non-strict ones (`>=`). This is due to some supplementary checks (ISZERO, 3 gas)  

I suggest using  `>=`  instead of `>` to avoid some opcodes here:

```solidity
vaults/NFTVault.sol:539:        return principal > calculatedDebt ? principal : calculatedDebt;
vaults/NFTVault.sol:775:        _amount = _amount > debtAmount ? debtAmount : _amount;
```

## An array's length should be cached to save gas in for-loops

Reading array length at each iteration of the loop takes 6 gas (3 for mload and 3 to place memory_offset) in the stack.  
  
Caching the array length in the stack saves around 3 gas per iteration.  

Here, I suggest storing the array's length in a variable before the for-loop, and use it instead:

```solidity
farming/LPFarming.sol:348:        for (uint256 i = 0; i < poolInfo.length; i++) {
vaults/yVault/strategies/StrategyPUSDConvex.sol:145:        for (uint256 i = 0; i < _strategyConfig.rewardTokens.length; i++) {
vaults/yVault/strategies/StrategyPUSDConvex.sol:319:            for (uint256 i = 0; i < rewardTokens.length; i++) {
vaults/NFTVault.sol:181:        for (uint256 i = 0; i < _typeInitializers.length; i++) {
vaults/NFTVault.sol:184:            for (uint256 j = 0; j < initializer.nfts.length; j++) {
```

This is already done here:

```solidity
farming/LPFarming.sol:281:        for (uint256 pid = 0; pid < length; ++pid) {
vaults/yVault/strategies/StrategyPUSDConvex.sol:231:        for (uint256 i = 0; i < length; i++) {
```

## `++i` costs less gas compared to `i++` or `i += 1`

`++i` costs less gas compared to `i++` or `i += 1` for unsigned integer, as pre-increment is cheaper (about 5 gas per iteration). This statement is true even with the optimizer enabled.

`i++` increments `i` and returns the initial value of `i`. Which means:  
  
```solidity
uint i = 1;  
i++; // == 1 but i == 2  
```
  
But `++i` returns the actual incremented value:  
  
```solidity
uint i = 1;  
++i; // == 2 and i == 2 too, so no need for a temporary variable  
```
  
In the first case, the compiler has to create a temporary variable (when used) for returning `1` instead of `2`  
  
Instances include:  

```solidity
farming/LPFarming.sol:348:        for (uint256 i = 0; i < poolInfo.length; i++) {
vaults/yVault/strategies/StrategyPUSDConvex.sol:145:        for (uint256 i = 0; i < _strategyConfig.rewardTokens.length; i++) {
vaults/yVault/strategies/StrategyPUSDConvex.sol:231:        for (uint256 i = 0; i < length; i++) {
vaults/yVault/strategies/StrategyPUSDConvex.sol:319:            for (uint256 i = 0; i < rewardTokens.length; i++) {
vaults/NFTVault.sol:181:        for (uint256 i = 0; i < _typeInitializers.length; i++) {
vaults/NFTVault.sol:184:            for (uint256 j = 0; j < initializer.nfts.length; j++) {
```

I suggest using `++i` instead of `i++` to increment the value of an uint variable.

This is already done here:

```solidity
farming/LPFarming.sol:281:        for (uint256 pid = 0; pid < length; ++pid) {
```

## Increments can be unchecked

In Solidity 0.8+, there's a default overflow check on unsigned integers. It's possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck cannot be made inline.  
  
[ethereum/solidity#10695](https://github.com/ethereum/solidity/issues/10695)

Instances include:  

```solidity
farming/LPFarming.sol:281:        for (uint256 pid = 0; pid < length; ++pid) {
farming/LPFarming.sol:348:        for (uint256 i = 0; i < poolInfo.length; i++) {
vaults/yVault/strategies/StrategyPUSDConvex.sol:145:        for (uint256 i = 0; i < _strategyConfig.rewardTokens.length; i++) {
vaults/yVault/strategies/StrategyPUSDConvex.sol:231:        for (uint256 i = 0; i < length; i++) {
vaults/yVault/strategies/StrategyPUSDConvex.sol:319:            for (uint256 i = 0; i < rewardTokens.length; i++) {
vaults/NFTVault.sol:181:        for (uint256 i = 0; i < _typeInitializers.length; i++) {
vaults/NFTVault.sol:184:            for (uint256 j = 0; j < initializer.nfts.length; j++) {
```

The code would go from:  
  
```solidity
for (uint256 i; i < numIterations; i++) {  
 // ...  
}  
```

to:  

```solidity
for (uint256 i; i < numIterations;) {  
 // ...  
 unchecked { ++i; }  
}  
```

The risk of overflow is inexistant for a `uint256` here.

## Use `calldata` instead of `memory`

When arguments are read-only on external functions, the data location should be `calldata`:

```solidity
contracts/vaults/NFTVault.sol:
  212:     function setDebtInterestApr(Rate memory _debtInterestApr) //@audit gas: should be calldata
  222:     function setValueIncreaseLockRate(Rate memory _valueIncreaseLockRate)  //@audit gas: should be calldata
  232:     function setCreditLimitRate(Rate memory _creditLimitRate)  //@audit gas: should be calldata
  247:     function setLiquidationLimitRate(Rate memory _liquidationLimitRate)  //@audit gas: should be calldata
  290:     function setOrganizationFeeRate(Rate memory _organizationFeeRate)  //@audit gas: should be calldata
  300:     function setInsurancePurchaseRate(Rate memory _insurancePurchaseRate)  //@audit gas: should be calldata
  311:         Rate memory _insuranceLiquidationPenaltyRate  //@audit gas: should be calldata
```

## Consider making some constants as non-public to save gas

Reducing from `public` to `private` or `internal` can save gas when a constant isn't used outside of its contract.
I suggest changing the visibility from `public` to `internal` or `private` here:

```solidity
tokens/JPEG.sol:10:    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
tokens/StableCoin.sol:22:    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
tokens/StableCoin.sol:23:    bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");
vaults/yVault/strategies/StrategyPUSDConvex.sol:66:    bytes32 public constant STRATEGIST_ROLE = keccak256("STRATEGIST_ROLE");
vaults/yVault/Controller.sol:15:    bytes32 public constant STRATEGIST_ROLE = keccak256("STRATEGIST_ROLE");
vaults/FungibleAssetVaultForDAO.sol:41:    bytes32 public constant WHITELISTED_ROLE = keccak256("WHITELISTED_ROLE");
vaults/NFTVault.sol:71:    bytes32 public constant DAO_ROLE = keccak256("DAO_ROLE");
vaults/NFTVault.sol:72:    bytes32 public constant LIQUIDATOR_ROLE = keccak256("LIQUIDATOR_ROLE");
vaults/NFTVault.sol:74:    bytes32 public constant CUSTOM_NFT_HASH = keccak256("CUSTOM");
```

## Public functions to external

The following functions could be set external to save gas and improve code quality.
External call cost is less expensive than of public functions.

```solidity
withdraw(IERC20,uint256) should be declared external:
 - Controller.withdraw(IERC20,uint256) (contracts/vaults/yVault/Controller.sol#151-154)
setFarmingPool(address) should be declared external:
 - YVault.setFarmingPool(address) (contracts/vaults/yVault/yVault.sol#115-118)
```

## Reduce the size of error messages (Long revert Strings)

Shortening revert strings to fit in 32 bytes will decrease deployment time gas and will decrease runtime gas when the revert condition is met.

Revert strings that are longer than 32 bytes require at least one additional mstore, along with additional overhead for computing memory offset, etc.

Revert strings > 32 bytes:

```solidity
tokens/JPEG.sol:23:            "JPEG: must have minter role to mint"
tokens/StableCoin.sol:41:            "StableCoin: must have minter role to mint"
tokens/StableCoin.sol:55:            "StableCoin: must have pauser role to pause"
tokens/StableCoin.sol:69:            "StableCoin: must have pauser role to unpause"
vaults/NFTVault.sol:394:            "credit_rate_exceeds_or_equals_liquidation_rate" 
```

I suggest shortening the revert strings to fit in 32 bytes, or using custom errors as described next.

## Use Custom Errors instead of Revert Strings to save Gas

Custom errors from Solidity 0.8.4 are cheaper than revert strings (cheaper deployment cost and runtime cost when the revert condition is met)

Source: <https://blog.soliditylang.org/2021/04/21/custom-errors/>:
> Starting from [Solidity v0.8.4](https://github.com/ethereum/solidity/releases/tag/v0.8.4), there is a convenient and gas-efficient way to explain to users why an operation failed through the use of custom errors. Until now, you could already use strings to give more information about failures (e.g., `revert("Insufficient funds.");`), but they are rather expensive, especially when it comes to deploy cost, and it is difficult to use dynamic information in them.

Custom errors are defined using the `error` statement, which can be used inside and outside of contracts (including interfaces and libraries).

Instances include:

```solidity
escrow/NFTEscrow.sol:18:        require(success, "FlashEscrow: call_failed");
escrow/NFTEscrow.sol:86:        require(
farming/LPFarming.sol:86:        require(
farming/LPFarming.sol:112:        require(_startBlock >= block.number, "Invalid start block");
farming/LPFarming.sol:113:        require(_endBlock > _startBlock, "Invalid end block");
farming/LPFarming.sol:114:        require(_rewardPerBlock > 0, "Invalid reward per block");
farming/LPFarming.sol:218:        require(_amount > 0, "invalid_amount");
farming/LPFarming.sol:239:        require(_amount > 0, "invalid_amount");
farming/LPFarming.sol:243:        require(user.amount >= _amount, "insufficient_amount");
farming/LPFarming.sol:337:        require(rewards > 0, "no_reward");
farming/LPFarming.sol:354:        require(rewards > 0, "no_reward");
farming/yVaultLPFarming.sol:42:        require(_vault != address(0), "INVALID_VAULT");
farming/yVaultLPFarming.sol:43:        require(_jpeg != address(0), "INVALID_JPEG");
farming/yVaultLPFarming.sol:55:        require(
farming/yVaultLPFarming.sol:101:        require(_amount > 0, "invalid_amount");
farming/yVaultLPFarming.sol:118:        require(_amount > 0, "invalid_amount");
farming/yVaultLPFarming.sol:119:        require(balanceOf[msg.sender] >= _amount, "insufficient_amount");
farming/yVaultLPFarming.sol:139:        require(rewards > 0, "no_reward");
helpers/CryptoPunksHelper.sol:79:        require(
helpers/EtherRocksHelper.sol:81:        require(
lock/JPEGLock.sol:40:        require(_newTime > 0, "Invalid lock time");
lock/JPEGLock.sol:70:        require(position.owner == msg.sender, "unauthorized");
lock/JPEGLock.sol:71:        require(position.unlockAt <= block.timestamp, "locked");
staking/JPEGStaking.sol:32:        require(_amount > 0, "invalid_amount");
staking/JPEGStaking.sol:45:        require(
tokens/JPEG.sol:21:        require(
tokens/StableCoin.sol:39:        require(
tokens/StableCoin.sol:53:        require(
tokens/StableCoin.sol:67:        require(
vaults/yVault/strategies/StrategyPUSDConvex.sol:108:        require(_want != address(0), "INVALID_WANT");
vaults/yVault/strategies/StrategyPUSDConvex.sol:109:        require(_jpeg != address(0), "INVALID_JPEG");
vaults/yVault/strategies/StrategyPUSDConvex.sol:110:        require(_pusd != address(0), "INVALID_PUSD");
vaults/yVault/strategies/StrategyPUSDConvex.sol:111:        require(_weth != address(0), "INVALID_WETH");
vaults/yVault/strategies/StrategyPUSDConvex.sol:112:        require(_usdc != address(0), "INVALID_USDC");
vaults/yVault/strategies/StrategyPUSDConvex.sol:113:        require(
vaults/yVault/strategies/StrategyPUSDConvex.sol:117:        require(
vaults/yVault/strategies/StrategyPUSDConvex.sol:121:        require(address(_curveConfig.curve) != address(0), "INVALID_CURVE");
vaults/yVault/strategies/StrategyPUSDConvex.sol:122:        require(
vaults/yVault/strategies/StrategyPUSDConvex.sol:126:        require(_curveConfig.pusdIndex < 4, "INVALID_PUSD_CURVE_INDEX");
vaults/yVault/strategies/StrategyPUSDConvex.sol:127:        require(_curveConfig.usdcIndex < 4, "INVALID_USDC_CURVE_INDEX");
vaults/yVault/strategies/StrategyPUSDConvex.sol:128:        require(
vaults/yVault/strategies/StrategyPUSDConvex.sol:132:        require(
vaults/yVault/strategies/StrategyPUSDConvex.sol:136:        require(
vaults/yVault/strategies/StrategyPUSDConvex.sol:140:        require(
vaults/yVault/strategies/StrategyPUSDConvex.sol:146:            require(
vaults/yVault/strategies/StrategyPUSDConvex.sol:168:        require(
vaults/yVault/strategies/StrategyPUSDConvex.sol:181:        require(
vaults/yVault/strategies/StrategyPUSDConvex.sol:195:        require(_controller != address(0), "INVALID_CONTROLLER");
vaults/yVault/strategies/StrategyPUSDConvex.sol:205:        require(_vault != address(0), "INVALID_USDC_VAULT");
vaults/yVault/strategies/StrategyPUSDConvex.sol:262:        require(want != _asset, "want");
vaults/yVault/strategies/StrategyPUSDConvex.sol:263:        require(pusd != _asset, "pusd");
vaults/yVault/strategies/StrategyPUSDConvex.sol:264:        require(usdc != _asset, "usdc");
vaults/yVault/strategies/StrategyPUSDConvex.sol:265:        require(weth != _asset, "weth");
vaults/yVault/strategies/StrategyPUSDConvex.sol:266:        require(jpeg != _asset, "jpeg");
vaults/yVault/strategies/StrategyPUSDConvex.sol:275:        require(vault != address(0), "ZERO_VAULT"); // additional protection so we don't burn the funds
vaults/yVault/strategies/StrategyPUSDConvex.sol:292:        require(vault != address(0), "ZERO_VAULT"); // additional protection so we don't burn the funds
vaults/yVault/strategies/StrategyPUSDConvex.sol:334:            require(wethBalance > 0, "NOOP");
vaults/yVault/Controller.sol:37:        require(_feeAddress != address(0), "INVALID_FEE_ADDRESS");
vaults/yVault/Controller.sol:48:        require(vaults[_token] == address(0), "ALREADY_HAS_VAULT");
vaults/yVault/Controller.sol:49:        require(_vault != address(0), "INVALID_VAULT");
vaults/yVault/Controller.sol:60:        require(address(_token) != address(0), "INVALID_TOKEN");
vaults/yVault/Controller.sol:61:        require(address(_strategy) != address(0), "INVALID_STRATEGY");
vaults/yVault/Controller.sol:73:        require(address(_token) != address(0), "INVALID_TOKEN");
vaults/yVault/Controller.sol:74:        require(address(_strategy) != address(0), "INVALID_STRATEGY");
vaults/yVault/Controller.sol:86:        require(
vaults/yVault/Controller.sol:152:        require(msg.sender == vaults[_token], "NOT_VAULT");
vaults/yVault/Controller.sol:164:        require(msg.sender == vaults[_token], "NOT_VAULT");
vaults/yVault/yVault.sol:62:        require(
vaults/yVault/yVault.sol:99:        require(
vaults/yVault/yVault.sol:109:        require(_controller != address(0), "INVALID_CONTROLLER");
vaults/yVault/yVault.sol:116:        require(_farm != address(0), "INVALID_FARMING_POOL");
vaults/yVault/yVault.sol:143:        require(_amount > 0, "INVALID_AMOUNT");
vaults/yVault/yVault.sol:167:        require(_shares > 0, "INVALID_AMOUNT");
vaults/yVault/yVault.sol:170:        require(supply > 0, "NO_TOKENS_DEPOSITED");
vaults/yVault/yVault.sol:188:        require(farm != address(0), "NO_FARM");
vaults/FungibleAssetVaultForDAO.sol:93:        require(
vaults/FungibleAssetVaultForDAO.sol:108:        require(answer > 0, "invalid_oracle_answer");
vaults/FungibleAssetVaultForDAO.sol:142:        require(amount > 0, "invalid_amount");
vaults/FungibleAssetVaultForDAO.sol:145:            require(msg.value == amount, "invalid_msg_value");
vaults/FungibleAssetVaultForDAO.sol:147:            require(msg.value == 0, "non_zero_eth_value");
vaults/FungibleAssetVaultForDAO.sol:164:        require(amount > 0, "invalid_amount");
vaults/FungibleAssetVaultForDAO.sol:168:        require(newDebtAmount <= creditLimit, "insufficient_credit");
vaults/FungibleAssetVaultForDAO.sol:180:        require(amount > 0, "invalid_amount");
vaults/FungibleAssetVaultForDAO.sol:194:        require(amount > 0 && amount <= collateralAmount, "invalid_amount");
vaults/FungibleAssetVaultForDAO.sol:197:        require(creditLimit >= debtAmount, "insufficient_credit");
vaults/NFTVault.sol:118:        require(nftContract.ownerOf(nftIndex) != address(0), "invalid_nft");
vaults/NFTVault.sol:278:        require(_newFloor > 0, "Invalid floor");
vaults/NFTVault.sol:326:        require(
vaults/NFTVault.sol:365:        require(pendingValue > 0, "no_pending_value");
vaults/NFTVault.sol:391:        require(
vaults/NFTVault.sol:401:        require(
vaults/NFTVault.sol:462:        require(answer > 0, "invalid_oracle_answer");
vaults/NFTVault.sol:682:        require(
vaults/NFTVault.sol:687:        require(_amount > 0, "invalid_amount");
vaults/NFTVault.sol:688:        require(
vaults/NFTVault.sol:698:        require(position.liquidatedAt == 0, "liquidated");
vaults/NFTVault.sol:699:        require(
vaults/NFTVault.sol:710:        require(debtAmount + _amount <= creditLimit, "insufficient_credit");
vaults/NFTVault.sol:763:        require(msg.sender == positionOwner[_nftIndex], "unauthorized");
vaults/NFTVault.sol:764:        require(_amount > 0, "invalid_amount");
vaults/NFTVault.sol:767:        require(position.liquidatedAt == 0, "liquidated");
vaults/NFTVault.sol:770:        require(debtAmount > 0, "position_not_borrowed");
vaults/NFTVault.sol:805:        require(msg.sender == positionOwner[_nftIndex], "unauthorized");
vaults/NFTVault.sol:806:        require(_getDebtAmount(_nftIndex) == 0, "position_not_repaid");
vaults/NFTVault.sol:839:        require(posOwner != address(0), "position_not_exist");
vaults/NFTVault.sol:842:        require(position.liquidatedAt == 0, "liquidated");
vaults/NFTVault.sol:845:        require(
vaults/NFTVault.sol:881:        require(msg.sender == positionOwner[_nftIndex], "unauthorized");
vaults/NFTVault.sol:882:        require(position.liquidatedAt > 0, "not_liquidated");
vaults/NFTVault.sol:883:        require(
vaults/NFTVault.sol:887:        require(
vaults/NFTVault.sol:925:        require(address(0) != owner, "no_position");
vaults/NFTVault.sol:926:        require(position.liquidatedAt > 0, "not_liquidated");
vaults/NFTVault.sol:927:        require(
vaults/NFTVault.sol:932:        require(position.liquidator == msg.sender, "unauthorized");
```

I suggest replacing revert strings with custom errors.
