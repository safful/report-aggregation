## Tags

- bug
- duplicate
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas Optimizations](https://github.com/code-423n4/2022-06-connext-findings/issues/261) 

**Overview**

Risk Rating | Number of issues
--- | ---
Gas Issues | 16

**Table of Contents:**

- [1. Avoid unnecessary calculation when `_args.amount == 0`](#1-avoid-unnecessary-calculation-when-_argsamount--0)
- [2. Cheap Contract Deployment Through Clones](#2-cheap-contract-deployment-through-clones)
- [3. Avoid emitting a storage variable when a memory value is available](#3-avoid-emitting-a-storage-variable-when-a-memory-value-is-available)
- [4. Reduce the size of error messages (Long revert Strings)](#4-reduce-the-size-of-error-messages-long-revert-strings)
- [5. SafeMath is not needed when using Solidity version 0.8+](#5-safemath-is-not-needed-when-using-solidity-version-08)
- [6. `>=` is cheaper than `>` (and `<=` cheaper than `<`)](#6--is-cheaper-than--and--cheaper-than-)
- [7. Splitting `require()` statements that use `&&` saves gas](#7-splitting-require-statements-that-use--saves-gas)
- [8. Using private rather than public for constants saves gas](#8-using-private-rather-than-public-for-constants-saves-gas)
- [9. `<array>.length` should not be looked up in every loop of a `for-loop`](#9-arraylength-should-not-be-looked-up-in-every-loop-of-a-for-loop)
- [10. `++i` costs less gas compared to `i++` or `i += 1` (same for `--i` vs `i--` or `i -= 1`)](#10-i-costs-less-gas-compared-to-i-or-i--1-same-for---i-vs-i---or-i---1)
- [11. Increments/decrements can be unchecked in for-loops](#11-incrementsdecrements-can-be-unchecked-in-for-loops)
- [12. It costs more gas to initialize variables with their default value than letting the default value be applied](#12-it-costs-more-gas-to-initialize-variables-with-their-default-value-than-letting-the-default-value-be-applied)
- [13. A variable should be immutable](#13-a-variable-should-be-immutable)
- [14. Use Custom Errors instead of Revert Strings to save Gas](#14-use-custom-errors-instead-of-revert-strings-to-save-gas)
- [15. Functions guaranteed to revert when called by normal users can be marked `payable`](#15-functions-guaranteed-to-revert-when-called-by-normal-users-can-be-marked-payable)
- [16. Use scientific notation (e.g. `1e18`) rather than exponentiation (e.g. `10**18`)](#16-use-scientific-notation-eg-1e18-rather-than-exponentiation-eg-1018)

## 1. Avoid unnecessary calculation when `_args.amount == 0`

[Here](https://github.com/code-423n4/2022-06-connext/blob/b4532655071566b33c41eac46e75be29b4a381ed/contracts/contracts/core/connext/facets/BridgeFacet.sol#L762), if `_args.amount == 0` (which is possible), there should be a return statement to avoid unnecessary gas consumption:

```solidity
File: BridgeFacet.sol
762:     uint256 toSwap = _args.amount; // @audit-info [INFO] amount can be 0 meaning that this should return to avoid unnecessary gas consumption. Recommendation: add a if (_args.amount == 0) return;
```

## 2. Cheap Contract Deployment Through Clones

```solidity
core/connext/facets/upgrade-initializers/DiamondInit.sol:73:      s.executor = new Executor(address(this));
```

There's a way to save a significant amount of gas on deployment using Clones: <https://www.youtube.com/watch?v=3Mw-pMmJ7TA> .

This is a solution that was adopted, as an example, by Porter Finance. They realized that deploying using clones was 10x cheaper:

- <https://github.com/porter-finance/v1-core/issues/15#issuecomment-1035639516>
- <https://github.com/porter-finance/v1-core/pull/34>

Consider applying a similar pattern.

## 3. Avoid emitting a storage variable when a memory value is available

When they are the same, consider emitting the memory value instead of the storage value:

```solidity
contracts/contracts/core/connext/helpers/ProposedOwnableUpgradeable.sol:
  320    function _setProposed(address newlyProposed) private {
  321      _proposedOwnershipTimestamp = block.timestamp;
  322      _proposed = newlyProposed;
  323:     emit OwnershipProposed(_proposed); //@audit should emit newlyProposed

contracts/contracts/core/shared/ProposedOwnable.sol:
  169    function _setProposed(address newlyProposed) private {
  170      _proposedOwnershipTimestamp = block.timestamp;
  171      _proposed = newlyProposed;
  172:     emit OwnershipProposed(_proposed);//@audit should emit newlyProposed
  173    }
```

## 4. Reduce the size of error messages (Long revert Strings)

Shortening revert strings to fit in 32 bytes will decrease deployment time gas and will decrease runtime gas when the revert condition is met.

Revert strings that are longer than 32 bytes require at least one additional mstore, along with additional overhead for computing memory offset, etc.

Revert strings > 32 bytes:

```solidity
core/connext/helpers/OZERC20.sol:185:    require(_sender != address(0), "ERC20: transfer from the zero address");
core/connext/helpers/OZERC20.sol:186:    require(_recipient != address(0), "ERC20: transfer to the zero address");
core/connext/helpers/OZERC20.sol:190:    balances[_sender] = balances[_sender].sub(amount, "ERC20: transfer amount exceeds balance");
core/connext/helpers/OZERC20.sol:226:    require(_account != address(0), "ERC20: burn from the zero address");
core/connext/helpers/OZERC20.sol:230:    balances[_account] = balances[_account].sub(_amount, "ERC20: burn amount exceeds balance");
core/connext/helpers/OZERC20.sol:253:    require(_owner != address(0), "ERC20: approve from the zero address");
core/connext/helpers/OZERC20.sol:254:    require(_spender != address(0), "ERC20: approve to the zero address");
core/connext/libraries/LibDiamond.sol:66:    require(msg.sender == diamondStorage().contractOwner, "LibDiamond: Must be contract owner");
core/connext/libraries/LibDiamond.sol:113:        revert("LibDiamondCut: Incorrect FacetCutAction");
core/connext/libraries/LibDiamond.sol:121:    require(_functionSelectors.length > 0, "LibDiamondCut: No selectors in facet to cut");
core/connext/libraries/LibDiamond.sol:123:    require(_facetAddress != address(0), "LibDiamondCut: Add facet can't be address(0)");
core/connext/libraries/LibDiamond.sol:132:      require(oldFacetAddress == address(0), "LibDiamondCut: Can't add function that already exists");
core/connext/libraries/LibDiamond.sol:139:    require(_functionSelectors.length > 0, "LibDiamondCut: No selectors in facet to cut");
core/connext/libraries/LibDiamond.sol:141:    require(_facetAddress != address(0), "LibDiamondCut: Add facet can't be address(0)");
core/connext/libraries/LibDiamond.sol:150:      require(oldFacetAddress != _facetAddress, "LibDiamondCut: Can't replace function with same function");
core/connext/libraries/LibDiamond.sol:158:    require(_functionSelectors.length > 0, "LibDiamondCut: No selectors in facet to cut");
core/connext/libraries/LibDiamond.sol:161:    require(_facetAddress == address(0), "LibDiamondCut: Remove facet address must be address(0)");
core/connext/libraries/LibDiamond.sol:170:    enforceHasContractCode(_facetAddress, "LibDiamondCut: New facet has no code");
core/connext/libraries/LibDiamond.sol:191:    require(_facetAddress != address(0), "LibDiamondCut: Can't remove function that doesn't exist");
core/connext/libraries/LibDiamond.sol:193:    require(_facetAddress != address(this), "LibDiamondCut: Can't remove immutable function");
core/connext/libraries/LibDiamond.sol:224:      require(_calldata.length == 0, "LibDiamondCut: _init is address(0) but_calldata is not empty");
core/connext/libraries/LibDiamond.sol:226:      require(_calldata.length > 0, "LibDiamondCut: _calldata is empty but _init is not address(0)");
core/connext/libraries/LibDiamond.sol:228:        enforceHasContractCode(_init, "LibDiamondCut: _init address has no code");
core/connext/libraries/LibDiamond.sol:236:          revert("LibDiamondCut: _init function reverted");
core/connext/libraries/SwapUtils.sol:595:        balances[i] = balances[i].sub(amounts[i], "Cannot withdraw more than available");
core/connext/libraries/SwapUtils.sol:697:    require(dy <= self.balances[tokenIndexTo], "Cannot get more than pool balance");
core/connext/libraries/SwapUtils.sol:784:    require(dy <= self.balances[tokenIndexTo], "Cannot get more than pool balance");
core/connext/libraries/SwapUtils.sol:1015:        balances1[i] = v.balances[i].sub(amounts[i], "Cannot withdraw more than available");
```

Consider shortening the revert strings to fit in 32 bytes.

## 5. SafeMath is not needed when using Solidity version 0.8+

Solidity version 0.8+ already implements overflow and underflow checks by default.
Using the SafeMath library from OpenZeppelin (which is more gas expensive than the 0.8+ overflow checks) is therefore redundant.

Consider using the built-in checks instead of SafeMath and remove SafeMath here:

```solidity
core/connext/helpers/ConnextPriceOracle.sol:2:pragma solidity 0.8.14;
core/connext/helpers/ConnextPriceOracle.sol:4:import {SafeMath} from "@openzeppelin/contracts/utils/math/SafeMath.sol";
core/connext/helpers/ConnextPriceOracle.sol:45:  using SafeMath for uint256;

core/connext/helpers/OZERC20.sol:2:pragma solidity 0.8.14;
core/connext/helpers/OZERC20.sol:10:import "@openzeppelin/contracts/utils/math/SafeMath.sol";
core/connext/helpers/OZERC20.sol:37:  using SafeMath for uint256;

core/connext/libraries/AmplificationUtils.sol:2:pragma solidity 0.8.14;
core/connext/libraries/AmplificationUtils.sol:5:import {SafeMath} from "@openzeppelin/contracts/utils/math/SafeMath.sol";
core/connext/libraries/AmplificationUtils.sol:15:  using SafeMath for uint256;

core/connext/libraries/SwapUtils.sol:2:pragma solidity 0.8.14;
core/connext/libraries/SwapUtils.sol:4:import {SafeMath} from "@openzeppelin/contracts/utils/math/SafeMath.sol";
core/connext/libraries/SwapUtils.sol:20:  using SafeMath for uint256;
```

## 6. `>=` is cheaper than `>` (and `<=` cheaper than `<`)

Strict inequalities (`>`) are more expensive than non-strict ones (`>=`). This is due to some supplementary checks (ISZERO, 3 gas). This also holds true between `<=` and `<`.  

Consider replacing strict inequalities with non-strict ones to save some gas here:

```solidity
core/connext/helpers/SponsorVault.sol:214:      sponsoredFee = balance < _liquidityFee ? balance : _liquidityFee;
core/connext/helpers/SponsorVault.sol:258:      sponsoredFee = sponsoredFee > address(this).balance ? address(this).balance : sponsoredFee;
```

## 7. Splitting `require()` statements that use `&&` saves gas

If you're using the Optimizer at 200, instead of using the `&&` operator in a single require statement to check multiple conditions, Consider using multiple require statements with 1 condition per require statement:

```solidity
core/connext/helpers/StableSwap.sol:85:          tokenIndexes[address(_pooledTokens[i])] == 0 && _pooledTokens[0] != _pooledTokens[i],
core/connext/libraries/AmplificationUtils.sol:86:    require(futureA_ > 0 && futureA_ < MAX_A, "futureA_ must be > 0 and < MAX_A");
core/connext/libraries/SwapUtils.sol:397:    require(tokenIndexFrom < numTokens && tokenIndexTo < numTokens, "Tokens must be in pool");
core/connext/libraries/SwapUtils.sol:493:    require(tokenIndexFrom < xp.length && tokenIndexTo < xp.length, "Token index out of range");
core/connext/libraries/SwapUtils.sol:524:    require(tokenIndexFrom < xp.length && tokenIndexTo < xp.length, "Token index out of range");
core/connext/libraries/SwapUtils.sol:1007:    require(maxBurnAmount <= v.lpToken.balanceOf(msg.sender) && maxBurnAmount != 0, ">LP.balanceOf");
```

Please, note that this might not hold true at a higher number of runs for the Optimizer (10k). However, it indeed is true at 200.

## 8. Using private rather than public for constants saves gas

If needed, the value can be read from the verified contract source code. Savings are due to the compiler not having to create non-payable getter functions for deployment calldata, and not adding another entry to the method ID table

```solidity
core/connext/facets/BridgeFacet.sol:68:  uint16 public constant AAVE_REFERRAL_CODE = 0;
core/connext/helpers/PriceOracle.sol:6:  bool public constant isPriceOracle = true;
core/connext/libraries/AmplificationUtils.sol:21:  uint256 public constant A_PRECISION = 100;
core/connext/libraries/AmplificationUtils.sol:22:  uint256 public constant MAX_A = 10**6;
core/connext/libraries/LibCrossDomainProperty.sol:37:  bytes29 public constant EMPTY = hex"ffffffffffffffffffffffffffffffffffffffffffffffffffffffffff";
core/connext/libraries/LibCrossDomainProperty.sol:38:  bytes public constant EMPTY_BYTES = hex"ffffffffffffffffffffffffffffffffffffffffffffffffffffffffff";
core/shared/Version.sol:9:  uint8 public constant VERSION = 0; 
```

## 9. `<array>.length` should not be looked up in every loop of a `for-loop`

Reading array length at each iteration of the loop consumes more gas than necessary.
  
In the best case scenario (length read on a memory variable), caching the array length in the stack saves around 3 gas per iteration.
In the worst case scenario (external calls at each iteration), the amount of gas wasted can be massive.

Here, Consider storing the array's length in a variable before the for-loop, and use this new variable instead:

```solidity
core/connext/facets/RelayerFacet.sol:140:    for (uint256 i; i < _transferIds.length; ) {
core/connext/facets/RelayerFacet.sol:164:    for (uint256 i; i < _transferIds.length; ) {
core/connext/facets/StableSwapFacet.sol:415:    for (uint8 i = 0; i < _pooledTokens.length; i++) {
core/connext/helpers/ConnextPriceOracle.sol:176:    for (uint256 i = 0; i < tokenAddresses.length; i++) {
core/connext/helpers/Multicall.sol:16:    for (uint256 i = 0; i < calls.length; i++) {
core/connext/helpers/StableSwap.sol:81:    for (uint8 i = 0; i < _pooledTokens.length; i++) {
core/connext/libraries/LibDiamond.sol:104:    for (uint256 facetIndex; facetIndex < _diamondCut.length; facetIndex++) {
core/connext/libraries/LibDiamond.sol:129:    for (uint256 selectorIndex; selectorIndex < _functionSelectors.length; selectorIndex++) {
core/connext/libraries/LibDiamond.sol:147:    for (uint256 selectorIndex; selectorIndex < _functionSelectors.length; selectorIndex++) {
core/connext/libraries/LibDiamond.sol:162:    for (uint256 selectorIndex; selectorIndex < _functionSelectors.length; selectorIndex++) {
core/connext/libraries/SwapUtils.sol:205:    for (uint256 i = 0; i < xp.length; i++) {
core/connext/libraries/SwapUtils.sol:558:    for (uint256 i = 0; i < balances.length; i++) {
core/connext/libraries/SwapUtils.sol:591:    for (uint256 i = 0; i < balances.length; i++) {
core/connext/libraries/SwapUtils.sol:844:    for (uint256 i = 0; i < pooledTokens.length; i++) {
core/connext/libraries/SwapUtils.sol:869:      for (uint256 i = 0; i < pooledTokens.length; i++) {
core/connext/libraries/SwapUtils.sol:924:    for (uint256 i = 0; i < amounts.length; i++) {
core/connext/libraries/SwapUtils.sol:1014:      for (uint256 i = 0; i < pooledTokens.length; i++) {
core/connext/libraries/SwapUtils.sol:1019:      for (uint256 i = 0; i < pooledTokens.length; i++) {
core/connext/libraries/SwapUtils.sol:1039:    for (uint256 i = 0; i < pooledTokens.length; i++) {
core/connext/libraries/SwapUtils.sol:1055:    for (uint256 i = 0; i < pooledTokens.length; i++) {
```

## 10. `++i` costs less gas compared to `i++` or `i += 1` (same for `--i` vs `i--` or `i -= 1`)

Pre-increments and pre-decrements are cheaper.

For a `uint256 i` variable, the following is true with the Optimizer enabled at 10k:

**Increment:**

- `i += 1` is the most expensive form
- `i++` costs 6 gas less than `i += 1`
- `++i` costs 5 gas less than `i++` (11 gas less than `i += 1`)

**Decrement:**

- `i -= 1` is the most expensive form
- `i--` costs 11 gas less than `i -= 1`
- `--i` costs 5 gas less than `i--` (16 gas less than `i -= 1`)

Note that post-increments (or post-decrements) return the old value before incrementing or decrementing, hence the name *post-increment*:

```solidity
uint i = 1;  
uint j = 2;
require(j == i++, "This will be false as i is incremented after the comparison");
```
  
However, pre-increments (or pre-decrements) return the new value:
  
```solidity
uint i = 1;  
uint j = 2;
require(j == ++i, "This will be true as i is incremented before the comparison");
```
  
In the pre-increment case, the compiler has to create a temporary variable (when used) for returning `1` instead of `2`.  
  
Affected code:  

```solidity
core/connext/facets/BridgeFacet.sol:332:      s.nonce += 1;
core/connext/facets/BridgeFacet.sol:613:          i++;
core/connext/facets/BridgeFacet.sol:684:          i++;
core/connext/facets/BridgeFacet.sol:799:            i++;
core/connext/facets/DiamondLoupeFacet.sol:31:    for (uint256 i; i < numFacets; i++) {
core/connext/facets/RelayerFacet.sol:144:        i++;
core/connext/facets/RelayerFacet.sol:168:        i++;
core/connext/facets/StableSwapFacet.sol:415:    for (uint8 i = 0; i < _pooledTokens.length; i++) {
core/connext/helpers/ConnextPriceOracle.sol:176:    for (uint256 i = 0; i < tokenAddresses.length; i++) {
core/connext/helpers/Multicall.sol:16:    for (uint256 i = 0; i < calls.length; i++) {
core/connext/helpers/StableSwap.sol:81:    for (uint8 i = 0; i < _pooledTokens.length; i++) {
core/connext/libraries/Encoding.sol:22:    for (uint8 i = 0; i < 10; i += 1) {
core/connext/libraries/Encoding.sol:36:    for (uint8 i = 31; i > 15; i -= 1) {
core/connext/libraries/Encoding.sol:45:      for (uint8 i = 15; i < 255; i -= 1) {
core/connext/libraries/LibDiamond.sol:104:    for (uint256 facetIndex; facetIndex < _diamondCut.length; facetIndex++) {
core/connext/libraries/LibDiamond.sol:129:    for (uint256 selectorIndex; selectorIndex < _functionSelectors.length; selectorIndex++) {
core/connext/libraries/LibDiamond.sol:134:      selectorPosition++;
core/connext/libraries/LibDiamond.sol:147:    for (uint256 selectorIndex; selectorIndex < _functionSelectors.length; selectorIndex++) {
core/connext/libraries/LibDiamond.sol:153:      selectorPosition++;
core/connext/libraries/LibDiamond.sol:162:    for (uint256 selectorIndex; selectorIndex < _functionSelectors.length; selectorIndex++) {
core/connext/libraries/SwapUtils.sol:205:    for (uint256 i = 0; i < xp.length; i++) {
core/connext/libraries/SwapUtils.sol:254:    for (uint256 i = 0; i < numTokens; i++) {
core/connext/libraries/SwapUtils.sol:268:    for (uint256 i = 0; i < MAX_LOOP_LIMIT; i++) {
core/connext/libraries/SwapUtils.sol:289:    for (uint256 i = 0; i < numTokens; i++) {
core/connext/libraries/SwapUtils.sol:300:    for (uint256 i = 0; i < MAX_LOOP_LIMIT; i++) {
core/connext/libraries/SwapUtils.sol:302:      for (uint256 j = 0; j < numTokens; j++) {
core/connext/libraries/SwapUtils.sol:344:    for (uint256 i = 0; i < numTokens; i++) {
core/connext/libraries/SwapUtils.sol:405:    for (uint256 i = 0; i < numTokens; i++) {
core/connext/libraries/SwapUtils.sol:425:    for (uint256 i = 0; i < MAX_LOOP_LIMIT; i++) {
core/connext/libraries/SwapUtils.sol:558:    for (uint256 i = 0; i < balances.length; i++) {
core/connext/libraries/SwapUtils.sol:591:    for (uint256 i = 0; i < balances.length; i++) {
core/connext/libraries/SwapUtils.sol:844:    for (uint256 i = 0; i < pooledTokens.length; i++) {
core/connext/libraries/SwapUtils.sol:869:      for (uint256 i = 0; i < pooledTokens.length; i++) {
core/connext/libraries/SwapUtils.sol:924:    for (uint256 i = 0; i < amounts.length; i++) {
core/connext/libraries/SwapUtils.sol:1014:      for (uint256 i = 0; i < pooledTokens.length; i++) {
core/connext/libraries/SwapUtils.sol:1019:      for (uint256 i = 0; i < pooledTokens.length; i++) {
core/connext/libraries/SwapUtils.sol:1039:    for (uint256 i = 0; i < pooledTokens.length; i++) {
core/connext/libraries/SwapUtils.sol:1055:    for (uint256 i = 0; i < pooledTokens.length; i++) {
core/relayer-fee/libraries/RelayerFeeMessage.sol:85:        i++;
```

Consider using pre-increments and pre-decrements where they are relevant (meaning: not where post-increments/decrements logic are relevant).

## 11. Increments/decrements can be unchecked in for-loops

In Solidity 0.8+, there's a default overflow check on unsigned integers. It's possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck cannot be made inline.  
  
[ethereum/solidity#10695](https://github.com/ethereum/solidity/issues/10695)

Affected code:  

```solidity
core/connext/facets/DiamondLoupeFacet.sol:31:    for (uint256 i; i < numFacets; i++) {
core/connext/facets/StableSwapFacet.sol:415:    for (uint8 i = 0; i < _pooledTokens.length; i++) {
core/connext/helpers/ConnextPriceOracle.sol:176:    for (uint256 i = 0; i < tokenAddresses.length; i++) {
core/connext/helpers/Multicall.sol:16:    for (uint256 i = 0; i < calls.length; i++) {
core/connext/helpers/StableSwap.sol:81:    for (uint8 i = 0; i < _pooledTokens.length; i++) {
core/connext/libraries/LibDiamond.sol:104:    for (uint256 facetIndex; facetIndex < _diamondCut.length; facetIndex++) {
core/connext/libraries/LibDiamond.sol:129:    for (uint256 selectorIndex; selectorIndex < _functionSelectors.length; selectorIndex++) {
core/connext/libraries/LibDiamond.sol:147:    for (uint256 selectorIndex; selectorIndex < _functionSelectors.length; selectorIndex++) {
core/connext/libraries/LibDiamond.sol:162:    for (uint256 selectorIndex; selectorIndex < _functionSelectors.length; selectorIndex++) {
core/connext/libraries/SwapUtils.sol:205:    for (uint256 i = 0; i < xp.length; i++) {
core/connext/libraries/SwapUtils.sol:254:    for (uint256 i = 0; i < numTokens; i++) {
core/connext/libraries/SwapUtils.sol:268:    for (uint256 i = 0; i < MAX_LOOP_LIMIT; i++) {
core/connext/libraries/SwapUtils.sol:289:    for (uint256 i = 0; i < numTokens; i++) {
core/connext/libraries/SwapUtils.sol:300:    for (uint256 i = 0; i < MAX_LOOP_LIMIT; i++) {
core/connext/libraries/SwapUtils.sol:302:      for (uint256 j = 0; j < numTokens; j++) {
core/connext/libraries/SwapUtils.sol:344:    for (uint256 i = 0; i < numTokens; i++) {
core/connext/libraries/SwapUtils.sol:405:    for (uint256 i = 0; i < numTokens; i++) {
core/connext/libraries/SwapUtils.sol:425:    for (uint256 i = 0; i < MAX_LOOP_LIMIT; i++) {
core/connext/libraries/SwapUtils.sol:558:    for (uint256 i = 0; i < balances.length; i++) {
core/connext/libraries/SwapUtils.sol:591:    for (uint256 i = 0; i < balances.length; i++) {
core/connext/libraries/SwapUtils.sol:844:    for (uint256 i = 0; i < pooledTokens.length; i++) {
core/connext/libraries/SwapUtils.sol:869:      for (uint256 i = 0; i < pooledTokens.length; i++) {
core/connext/libraries/SwapUtils.sol:924:    for (uint256 i = 0; i < amounts.length; i++) {
core/connext/libraries/SwapUtils.sol:1014:      for (uint256 i = 0; i < pooledTokens.length; i++) {
core/connext/libraries/SwapUtils.sol:1019:      for (uint256 i = 0; i < pooledTokens.length; i++) {
core/connext/libraries/SwapUtils.sol:1039:    for (uint256 i = 0; i < pooledTokens.length; i++) {
core/connext/libraries/SwapUtils.sol:1055:    for (uint256 i = 0; i < pooledTokens.length; i++) {
```

The change would be:  
  
```diff
- for (uint256 i; i < numIterations; i++) {
+ for (uint256 i; i < numIterations;) {
 // ...  
+   unchecked { ++i; }
}  
```

The same can be applied with decrements (which should use `break` when `i == 0`).

The risk of overflow is non-existant for `uint256` here.

## 12. It costs more gas to initialize variables with their default value than letting the default value be applied

If a variable is not set/initialized, it is assumed to have the default value (`0` for `uint`, `false` for `bool`, `address(0)` for address...). Explicitly initializing it with its default value is an anti-pattern and wastes gas.

As an example: `for (uint256 i = 0; i < numIterations; ++i) {` should be replaced with `for (uint256 i; i < numIterations; ++i) {`

Affected code:

```solidity
core/connext/facets/StableSwapFacet.sol:415:    for (uint8 i = 0; i < _pooledTokens.length; i++) {
core/connext/helpers/ConnextPriceOracle.sol:176:    for (uint256 i = 0; i < tokenAddresses.length; i++) {
core/connext/helpers/Multicall.sol:16:    for (uint256 i = 0; i < calls.length; i++) {
core/connext/helpers/StableSwap.sol:81:    for (uint8 i = 0; i < _pooledTokens.length; i++) {
core/connext/libraries/Encoding.sol:22:    for (uint8 i = 0; i < 10; i += 1) {
core/connext/libraries/SwapUtils.sol:205:    for (uint256 i = 0; i < xp.length; i++) {
core/connext/libraries/SwapUtils.sol:254:    for (uint256 i = 0; i < numTokens; i++) {
core/connext/libraries/SwapUtils.sol:268:    for (uint256 i = 0; i < MAX_LOOP_LIMIT; i++) {
core/connext/libraries/SwapUtils.sol:289:    for (uint256 i = 0; i < numTokens; i++) {
core/connext/libraries/SwapUtils.sol:300:    for (uint256 i = 0; i < MAX_LOOP_LIMIT; i++) {
core/connext/libraries/SwapUtils.sol:302:      for (uint256 j = 0; j < numTokens; j++) {
core/connext/libraries/SwapUtils.sol:344:    for (uint256 i = 0; i < numTokens; i++) {
core/connext/libraries/SwapUtils.sol:405:    for (uint256 i = 0; i < numTokens; i++) {
core/connext/libraries/SwapUtils.sol:425:    for (uint256 i = 0; i < MAX_LOOP_LIMIT; i++) {
core/connext/libraries/SwapUtils.sol:558:    for (uint256 i = 0; i < balances.length; i++) {
core/connext/libraries/SwapUtils.sol:591:    for (uint256 i = 0; i < balances.length; i++) {
core/connext/libraries/SwapUtils.sol:844:    for (uint256 i = 0; i < pooledTokens.length; i++) {
core/connext/libraries/SwapUtils.sol:869:      for (uint256 i = 0; i < pooledTokens.length; i++) {
core/connext/libraries/SwapUtils.sol:924:    for (uint256 i = 0; i < amounts.length; i++) {
core/connext/libraries/SwapUtils.sol:1014:      for (uint256 i = 0; i < pooledTokens.length; i++) {
core/connext/libraries/SwapUtils.sol:1019:      for (uint256 i = 0; i < pooledTokens.length; i++) {
core/connext/libraries/SwapUtils.sol:1039:    for (uint256 i = 0; i < pooledTokens.length; i++) {
core/connext/libraries/SwapUtils.sol:1055:    for (uint256 i = 0; i < pooledTokens.length; i++) {
core/relayer-fee/libraries/RelayerFeeMessage.sol:81:    for (uint256 i = 0; i < length; ) {
```

Consider removing explicit initializations for default values.

## 13. A variable should be immutable

This variable is only set in the constructor and never edited after that:

```solidity
core/connext/helpers/ConnextPriceOracle.sol:49:  address public wrapped;
```

Consider marking it as immutable, as it would avoid the expensive storage-writing operation (around 20 000 gas)

## 14. Use Custom Errors instead of Revert Strings to save Gas

Solidity 0.8.4 introduced custom errors. They are more gas efficient than revert strings, when it comes to deploy cost as well as runtime cost when the revert condition is met. Use custom errors instead of revert strings for gas savings.

Custom errors from Solidity 0.8.4 are cheaper than revert strings (cheaper deployment cost and runtime cost when the revert condition is met)

Source: <https://blog.soliditylang.org/2021/04/21/custom-errors/>:
> Starting from [Solidity v0.8.4](https://github.com/ethereum/solidity/releases/tag/v0.8.4), there is a convenient and gas-efficient way to explain to users why an operation failed through the use of custom errors. Until now, you could already use strings to give more information about failures (e.g., `revert("Insufficient funds.");`), but they are rather expensive, especially when it comes to deploy cost, and it is difficult to use dynamic information in them.

Custom errors are defined using the `error` statement, which can be used inside and outside of contracts (including interfaces and libraries).

Consider replacing all revert strings with custom errors in the solution.

```solidity
core/connext/facets/BaseConnextFacet.sol:38:    require(s._status != _ENTERED, "ReentrancyGuard: reentrant call");
core/connext/facets/BaseConnextFacet.sol:125:    require(_remote != bytes32(0), "!remote");
core/connext/helpers/BridgeToken.sol:94:    require(
core/connext/helpers/BridgeToken.sol:130:    require(block.timestamp <= _deadline, "ERC20Permit: expired deadline");
core/connext/helpers/BridgeToken.sol:131:    require(_owner != address(0), "ERC20Permit: owner zero address");
core/connext/helpers/BridgeToken.sol:136:    require(_signer == _owner, "ERC20Permit: invalid signature");
core/connext/helpers/ConnextPriceOracle.sol:72:    require(msg.sender == admin, "caller is not the admin");
core/connext/helpers/ConnextPriceOracle.sol:150:    require(baseTokenPrice > 0, "invalid base token");
core/connext/helpers/Executor.sol:57:    require(msg.sender == connext, "#OC:027");
core/connext/helpers/LPToken.sol:35:    require(amount != 0, "LPToken: cannot mint 0");
core/connext/helpers/LPToken.sol:50:    require(to != address(this), "LPToken: cannot send to itself");
core/connext/helpers/Multicall.sol:18:      require(success);
core/connext/helpers/OZERC20.sol:185:    require(_sender != address(0), "ERC20: transfer from the zero address");
core/connext/helpers/OZERC20.sol:186:    require(_recipient != address(0), "ERC20: transfer to the zero address");
core/connext/helpers/OZERC20.sol:205:    require(_account != address(0), "ERC20: mint to the zero address");
core/connext/helpers/OZERC20.sol:226:    require(_account != address(0), "ERC20: burn from the zero address");
core/connext/helpers/OZERC20.sol:253:    require(_owner != address(0), "ERC20: approve from the zero address");
core/connext/helpers/OZERC20.sol:254:    require(_spender != address(0), "ERC20: approve to the zero address");
core/connext/helpers/StableSwap.sol:75:    require(_pooledTokens.length > 1, "_pooledTokens.length <= 1");
core/connext/helpers/StableSwap.sol:76:    require(_pooledTokens.length <= 32, "_pooledTokens.length > 32");
core/connext/helpers/StableSwap.sol:77:    require(_pooledTokens.length == decimals.length, "_pooledTokens decimals mismatch");
core/connext/helpers/StableSwap.sol:84:        require(
core/connext/helpers/StableSwap.sol:89:      require(address(_pooledTokens[i]) != address(0), "The 0 address isn't an ERC-20");
core/connext/helpers/StableSwap.sol:90:      require(decimals[i] <= SwapUtils.POOL_PRECISION_DECIMALS, "Token decimals exceeds max");
core/connext/helpers/StableSwap.sol:96:    require(_a < AmplificationUtils.MAX_A, "_a exceeds maximum");
core/connext/helpers/StableSwap.sol:97:    require(_fee < SwapUtils.MAX_SWAP_FEE, "_fee exceeds maximum");
core/connext/helpers/StableSwap.sol:98:    require(_adminFee < SwapUtils.MAX_ADMIN_FEE, "_adminFee exceeds maximum");
core/connext/helpers/StableSwap.sol:102:    require(lpToken.initialize(lpTokenName, lpTokenSymbol), "could not init lpToken clone");
core/connext/helpers/StableSwap.sol:125:    require(block.timestamp <= deadline, "Deadline not met");
core/connext/helpers/StableSwap.sol:155:    require(index < swapStorage.pooledTokens.length, "Out of range");
core/connext/helpers/StableSwap.sol:167:    require(address(getToken(index)) == tokenAddress, "Token does not exist");
core/connext/helpers/StableSwap.sol:177:    require(index < swapStorage.pooledTokens.length, "Index out of range");
core/connext/helpers/TokenRegistry.sol:163:    require(_tokenId.domain != 0, "!repr");
core/connext/helpers/TokenRegistry.sol:228:    require(_token != address(0), "!token");
core/connext/libraries/AmplificationUtils.sol:84:    require(block.timestamp >= self.initialATime.add(1 days), "Wait 1 day before starting ramp");
core/connext/libraries/AmplificationUtils.sol:85:    require(futureTime_ >= block.timestamp.add(MIN_RAMP_TIME), "Insufficient ramp time");
core/connext/libraries/AmplificationUtils.sol:86:    require(futureA_ > 0 && futureA_ < MAX_A, "futureA_ must be > 0 and < MAX_A");
core/connext/libraries/AmplificationUtils.sol:92:      require(futureAPrecise.mul(MAX_A_CHANGE) >= initialAPrecise, "futureA_ is too small");
core/connext/libraries/AmplificationUtils.sol:94:      require(futureAPrecise <= initialAPrecise.mul(MAX_A_CHANGE), "futureA_ is too large");
core/connext/libraries/AmplificationUtils.sol:111:    require(self.futureATime > block.timestamp, "Ramp is already stopped");
core/connext/libraries/ConnextMessage.sol:116:    require(isValidAction(_action), "!action");
core/connext/libraries/LibDiamond.sol:66:    require(msg.sender == diamondStorage().contractOwner, "LibDiamond: Must be contract owner");
core/connext/libraries/LibDiamond.sol:100:    require(
core/connext/libraries/LibDiamond.sol:121:    require(_functionSelectors.length > 0, "LibDiamondCut: No selectors in facet to cut");
core/connext/libraries/LibDiamond.sol:123:    require(_facetAddress != address(0), "LibDiamondCut: Add facet can't be address(0)");
core/connext/libraries/LibDiamond.sol:132:      require(oldFacetAddress == address(0), "LibDiamondCut: Can't add function that already exists");
core/connext/libraries/LibDiamond.sol:139:    require(_functionSelectors.length > 0, "LibDiamondCut: No selectors in facet to cut");
core/connext/libraries/LibDiamond.sol:141:    require(_facetAddress != address(0), "LibDiamondCut: Add facet can't be address(0)");
core/connext/libraries/LibDiamond.sol:150:      require(oldFacetAddress != _facetAddress, "LibDiamondCut: Can't replace function with same function");
core/connext/libraries/LibDiamond.sol:158:    require(_functionSelectors.length > 0, "LibDiamondCut: No selectors in facet to cut");
core/connext/libraries/LibDiamond.sol:161:    require(_facetAddress == address(0), "LibDiamondCut: Remove facet address must be address(0)");
core/connext/libraries/LibDiamond.sol:191:    require(_facetAddress != address(0), "LibDiamondCut: Can't remove function that doesn't exist");
core/connext/libraries/LibDiamond.sol:193:    require(_facetAddress != address(this), "LibDiamondCut: Can't remove immutable function");
core/connext/libraries/LibDiamond.sol:224:      require(_calldata.length == 0, "LibDiamondCut: _init is address(0) but_calldata is not empty");
core/connext/libraries/LibDiamond.sol:226:      require(_calldata.length > 0, "LibDiamondCut: _calldata is empty but _init is not address(0)");
core/connext/libraries/LibDiamond.sol:247:    require(contractSize > 0, _errorMessage);
core/connext/libraries/SwapUtils.sol:191:    require(tokenIndex < xp.length, "Token index out of range");
core/connext/libraries/SwapUtils.sol:198:    require(tokenAmount <= xp[tokenIndex], "Withdraw exceeds available");
core/connext/libraries/SwapUtils.sol:248:    require(tokenIndex < numTokens, "Token not found");
core/connext/libraries/SwapUtils.sol:342:    require(numTokens == precisionMultipliers.length, "Balances must match multipliers");
core/connext/libraries/SwapUtils.sol:396:    require(tokenIndexFrom != tokenIndexTo, "Can't compare token to itself");
core/connext/libraries/SwapUtils.sol:397:    require(tokenIndexFrom < numTokens && tokenIndexTo < numTokens, "Tokens must be in pool");
core/connext/libraries/SwapUtils.sol:493:    require(tokenIndexFrom < xp.length && tokenIndexTo < xp.length, "Token index out of range");
core/connext/libraries/SwapUtils.sol:524:    require(tokenIndexFrom < xp.length && tokenIndexTo < xp.length, "Token index out of range");
core/connext/libraries/SwapUtils.sol:554:    require(amount <= totalSupply, "Cannot exceed total supply");
core/connext/libraries/SwapUtils.sol:615:    require(index < self.pooledTokens.length, "Token index out of range");
core/connext/libraries/SwapUtils.sol:649:      require(dx <= tokenFrom.balanceOf(msg.sender), "Cannot swap more than you own");
core/connext/libraries/SwapUtils.sol:662:    require(dy >= minDy, "Swap didn't result in min tokens");
core/connext/libraries/SwapUtils.sol:697:    require(dy <= self.balances[tokenIndexTo], "Cannot get more than pool balance");
core/connext/libraries/SwapUtils.sol:703:    require(dx <= maxDx, "Swap needs more than max tokens");
core/connext/libraries/SwapUtils.sol:717:      require(dx <= tokenFrom.balanceOf(msg.sender), "Cannot swap more than you own");
core/connext/libraries/SwapUtils.sol:723:      require(dx == tokenFrom.balanceOf(address(this)).sub(beforeBalance), "not support fee token");
core/connext/libraries/SwapUtils.sol:750:    require(dx <= tokenFrom.balanceOf(msg.sender), "Cannot swap more than you own");
core/connext/libraries/SwapUtils.sol:756:    require(dy >= minDy, "Swap didn't result in min tokens");
core/connext/libraries/SwapUtils.sol:784:    require(dy <= self.balances[tokenIndexTo], "Cannot get more than pool balance");
core/connext/libraries/SwapUtils.sol:790:    require(dx <= maxDx, "Swap didn't result in min tokens");
core/connext/libraries/SwapUtils.sol:823:    require(amounts.length == pooledTokens.length, "Amounts must match pooled tokens");
core/connext/libraries/SwapUtils.sol:845:      require(v.totalSupply != 0 || amounts[i] > 0, "Must supply all tokens in pool");
core/connext/libraries/SwapUtils.sol:861:    require(v.d1 > v.d0, "D should increase");
core/connext/libraries/SwapUtils.sol:890:    require(toMint >= minToMint, "Couldn't mint min requested");
core/connext/libraries/SwapUtils.sol:916:    require(amount <= lpToken.balanceOf(msg.sender), ">LP.balanceOf");
core/connext/libraries/SwapUtils.sol:917:    require(minAmounts.length == pooledTokens.length, "minAmounts must match poolTokens");
core/connext/libraries/SwapUtils.sol:925:      require(amounts[i] >= minAmounts[i], "amounts[i] < minAmounts[i]");
core/connext/libraries/SwapUtils.sol:954:    require(tokenAmount <= lpToken.balanceOf(msg.sender), ">LP.balanceOf");
core/connext/libraries/SwapUtils.sol:955:    require(tokenIndex < pooledTokens.length, "Token not found");
core/connext/libraries/SwapUtils.sol:961:    require(dy >= minAmount, "dy < minAmount");
core/connext/libraries/SwapUtils.sol:1005:    require(amounts.length == pooledTokens.length, "Amounts should match pool tokens");
core/connext/libraries/SwapUtils.sol:1007:    require(maxBurnAmount <= v.lpToken.balanceOf(msg.sender) && maxBurnAmount != 0, ">LP.balanceOf");
core/connext/libraries/SwapUtils.sol:1032:    require(tokenAmount != 0, "Burnt amount cannot be zero");
core/connext/libraries/SwapUtils.sol:1035:    require(tokenAmount <= maxBurnAmount, "tokenAmount > maxBurnAmount");
core/connext/libraries/SwapUtils.sol:1071:    require(newAdminFee <= MAX_ADMIN_FEE, "Fee is too high");
core/connext/libraries/SwapUtils.sol:1084:    require(newSwapFee <= MAX_SWAP_FEE, "Fee is too high");
core/shared/Router.sol:23:    require(_isRemoteRouter(_origin, _router), "!remote router");
core/shared/Router.sol:64:    require(_remote != bytes32(0), "!remote");
core/shared/XAppConnectionClient.sol:22:    require(_isReplica(msg.sender), "!replica");
```

## 15. Functions guaranteed to revert when called by normal users can be marked `payable`

If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

```solidity
core/connext/facets/AssetFacet.sol:100:  function setWrapper(address _wrapper) external onlyOwner {
core/connext/facets/AssetFacet.sol:112:  function setTokenRegistry(address _tokenRegistry) external onlyOwner {
core/connext/facets/AssetFacet.sol:162:  function addStableSwapPool(ConnextMessage.TokenId calldata _canonical, address _stableSwapPool) external onlyOwner {
core/connext/facets/AssetFacet.sol:171:  function removeAssetId(bytes32 _canonicalId, address _adoptedAssetId) external onlyOwner {
core/connext/facets/BridgeFacet.sol:233:  function setPromiseRouter(address payable _promiseRouter) external onlyOwner {
core/connext/facets/BridgeFacet.sol:242:  function setExecutor(address _executor) external onlyOwner {
core/connext/facets/BridgeFacet.sol:250:  function setSponsorVault(address _sponsorVault) external onlyOwner {
core/connext/facets/NomadFacet.sol:25:  function setXAppConnectionManager(address _xAppConnectionManager) external onlyOwner {
core/connext/facets/NomadFacet.sol:34:  function enrollRemoteRouter(uint32 _domain, bytes32 _router) external onlyOwner {
core/connext/facets/PortalFacet.sol:57:  function setAavePool(address _aavePool) external onlyOwner {
core/connext/facets/PortalFacet.sol:65:  function setAavePortalFee(uint256 _aavePortalFeeNumerator) external onlyOwner {
core/connext/facets/ProposedOwnableFacet.sol:128:  function proposeRouterOwnershipRenunciation() public onlyOwner {
core/connext/facets/ProposedOwnableFacet.sol:142:  function renounceRouterOwnership() public onlyOwner {
core/connext/facets/ProposedOwnableFacet.sol:162:  function proposeAssetOwnershipRenunciation() public onlyOwner {
core/connext/facets/ProposedOwnableFacet.sol:175:  function renounceAssetOwnership() public onlyOwner {
core/connext/facets/ProposedOwnableFacet.sol:203:  function proposeNewOwner(address newlyProposed) public onlyOwner {
core/connext/facets/ProposedOwnableFacet.sol:217:  function renounceOwnership() public onlyOwner {
core/connext/facets/ProposedOwnableFacet.sol:236:  function acceptProposedOwner() public onlyProposed {
core/connext/facets/ProposedOwnableFacet.sol:253:  function pause() public onlyOwner {
core/connext/facets/ProposedOwnableFacet.sol:258:  function unpause() public onlyOwner {
core/connext/facets/RelayerFacet.sol:88:  function setRelayerFeeRouter(address _relayerFeeRouter) external onlyOwner {
core/connext/facets/RelayerFacet.sol:101:  function addRelayer(address _relayer) external onlyOwner {
core/connext/facets/RelayerFacet.sol:112:  function removeRelayer(address _relayer) external onlyOwner {
core/connext/facets/RelayerFacet.sol:161:  function claim(address _recipient, bytes32[] calldata _transferIds) external onlyRelayerFeeRouter {
core/connext/facets/RoutersFacet.sol:293:  function removeRouter(address router) external onlyOwner {
core/connext/facets/RoutersFacet.sol:331:  function setMaxRoutersPerTransfer(uint256 _newMaxRouters) external onlyOwner {
core/connext/facets/RoutersFacet.sol:345:  function setLiquidityFeeNumerator(uint256 _numerator) external onlyOwner {
core/connext/facets/RoutersFacet.sol:361:  function approveRouterForPortal(address _router) external onlyOwner {
core/connext/facets/RoutersFacet.sol:375:  function unapproveRouterForPortal(address _router) external onlyOwner {
core/connext/facets/RoutersFacet.sol:393:  function setRouterRecipient(address router, address recipient) external onlyRouterOwner(router) {
core/connext/facets/RoutersFacet.sol:410:  function proposeRouterOwner(address router, address proposed) external onlyRouterOwner(router) {
core/connext/facets/RoutersFacet.sol:430:  function acceptProposedRouterOwner(address router) external onlyProposedRouterOwner(router) {
core/connext/facets/StableSwapFacet.sol:460:  function withdrawSwapAdminFees(bytes32 canonicalId) external onlyOwner {
core/connext/facets/StableSwapFacet.sol:469:  function setSwapAdminFee(bytes32 canonicalId, uint256 newAdminFee) external onlyOwner {
core/connext/facets/StableSwapFacet.sol:478:  function setSwapFee(bytes32 canonicalId, uint256 newSwapFee) external onlyOwner {
core/connext/facets/StableSwapFacet.sol:502:  function stopRampA(bytes32 canonicalId) external onlyOwner {
core/connext/helpers/BridgeToken.sol:54:  function burn(address _from, uint256 _amnt) external override onlyOwner {
core/connext/helpers/BridgeToken.sol:66:  function mint(address _to, uint256 _amnt) external override onlyOwner {
core/connext/helpers/BridgeToken.sol:73:  function setDetailsHash(bytes32 _detailsHash) external override onlyOwner {
core/connext/helpers/BridgeToken.sol:202:  function transferOwnership(address _newOwner) public override(IBridgeToken, OwnableUpgradeable) onlyOwner {
core/connext/helpers/ConnextPriceOracle.sol:158:  function setDirectPrice(address _token, uint256 _price) external onlyAdmin {
core/connext/helpers/ConnextPriceOracle.sol:163:  function setV1PriceOracle(address _v1PriceOracle) external onlyAdmin {
core/connext/helpers/ConnextPriceOracle.sol:168:  function setAdmin(address newAdmin) external onlyAdmin {
core/connext/helpers/ConnextPriceOracle.sol:175:  function setAggregators(address[] calldata tokenAddresses, address[] calldata sources) external onlyAdmin {
core/connext/helpers/LPToken.sol:34:  function mint(address recipient, uint256 amount) external onlyOwner {
core/connext/helpers/OwnerPausableUpgradeable.sol:14:  function __OwnerPausable_init() internal onlyInitializing {
core/connext/helpers/OwnerPausableUpgradeable.sol:23:  function pause() external onlyOwner {
core/connext/helpers/OwnerPausableUpgradeable.sol:30:  function unpause() external onlyOwner {
core/connext/helpers/OZERC20.sol:263:   * WARNING: This function should only be called from the constructor. Most
core/connext/helpers/ProposedOwnableUpgradeable.sol:77:  function __ProposedOwnable_init() internal onlyInitializing {
core/connext/helpers/ProposedOwnableUpgradeable.sol:81:  function __ProposedOwnable_init_unchained() internal onlyInitializing {
core/connext/helpers/ProposedOwnableUpgradeable.sol:155:  function proposeRouterOwnershipRenunciation() public virtual onlyOwner {
core/connext/helpers/ProposedOwnableUpgradeable.sol:169:  function renounceRouterOwnership() public virtual onlyOwner {
core/connext/helpers/ProposedOwnableUpgradeable.sol:197:  function proposeAssetOwnershipRenunciation() public virtual onlyOwner {
core/connext/helpers/ProposedOwnableUpgradeable.sol:211:  function renounceAssetOwnership() public virtual onlyOwner {
core/connext/helpers/ProposedOwnableUpgradeable.sol:239:  function proposeNewOwner(address newlyProposed) public virtual onlyOwner {
core/connext/helpers/ProposedOwnableUpgradeable.sol:253:  function renounceOwnership() public virtual onlyOwner {
core/connext/helpers/ProposedOwnableUpgradeable.sol:272:  function acceptProposedOwner() public virtual onlyProposed {
core/connext/helpers/SponsorVault.sol:138:  function setConnext(address _connext) external onlyOwner {
core/connext/helpers/SponsorVault.sol:147:  function setRate(uint32 _originDomain, Rate calldata _rate) external onlyOwner {
core/connext/helpers/SponsorVault.sol:159:  function setRelayerFeeCap(uint256 _relayerFeeCap) external onlyOwner {
core/connext/helpers/SponsorVault.sol:168:  function setGasTokenOracle(address _gasTokenOracle) external onlyOwner {
core/connext/helpers/SponsorVault.sol:178:  function setTokenExchange(address _token, address payable _tokenExchange) external onlyOwner {
core/connext/helpers/StableSwap.sol:440:  function withdrawAdminFees() external onlyOwner {
core/connext/helpers/StableSwap.sol:448:  function setAdminFee(uint256 newAdminFee) external onlyOwner {
core/connext/helpers/StableSwap.sol:456:  function setSwapFee(uint256 newSwapFee) external onlyOwner {
core/connext/helpers/StableSwap.sol:467:  function rampA(uint256 futureA, uint256 futureTime) external onlyOwner {
core/connext/helpers/StableSwap.sol:474:  function stopRampA() external onlyOwner {
core/promise/PromiseRouter.sol:155:  function setConnext(address _connext) external onlyOwner {
core/relayer-fee/RelayerFeeRouter.sol:89:  function setConnext(address _connext) external onlyOwner {
core/shared/ProposedOwnable.sol:109:  function proposeNewOwner(address newlyProposed) public virtual onlyOwner {
core/shared/ProposedOwnable.sol:123:  function renounceOwnership() public virtual onlyOwner {
core/shared/ProposedOwnable.sol:142:  function acceptProposedOwner() public virtual onlyProposed {
core/shared/ProposedOwnable.sol:180:  function __ProposedOwnable_init() internal onlyInitializing {
core/shared/ProposedOwnable.sol:184:  function __ProposedOwnable_init_unchained() internal onlyInitializing {
core/shared/Router.sol:34:  function enrollRemoteRouter(uint32 _domain, bytes32 _router) external onlyOwner {
core/shared/XAppConnectionClient.sol:39:  function setXAppConnectionManager(address _xAppConnectionManager) external onlyOwner {
```

## 16. Use scientific notation (e.g. `1e18`) rather than exponentiation (e.g. `10**18`)

```solidity
core/connext/libraries/SwapUtils.sol:104:  uint256 internal constant FEE_DENOMINATOR = 10**10;
core/connext/libraries/SwapUtils.sol:107:  uint256 internal constant MAX_SWAP_FEE = 10**8;
core/connext/libraries/SwapUtils.sol:113:  uint256 internal constant MAX_ADMIN_FEE = 10**10;
```
