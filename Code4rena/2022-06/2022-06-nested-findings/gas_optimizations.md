## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed
- valid

# [Gas Optimizations](https://github.com/code-423n4/2022-06-nested-findings/issues/82) 

# Gas Report

## Table of Contents

- [Caching storage variables in memory to save gas](#caching-storage-variables-in-memory-to-save-gas)
- [Calldata instead of memory for RO function parameters](#calldata-instead-of-memory-for-ro-function-parameters)
- [Comparison operators](#comparison-operators)
- [Constructor parameters should be avoided when possible](#constructor-parameters-should-be-avoided-when-possible)
- [Default value initialization](#default-value-initialization)
- [Mathematical optimizations](#mathematical-optimizations)
- [Require instead of AND](#require-instead-of-and)
- [Revert strings length](#revert-strings-length)
- [Shifting cheaper than division](#shifting-cheaper-than-division)
- [Tight variable packing](#tight-variable-packing)
- [unchecked arithmetic](#unchecked-arithmetic)
- [unnecessary computation](#unnecessary-computation)

# Caching storage variables in memory to save gas

## IMPACT

Anytime you are reading from storage more than once, it is cheaper in gas cost to cache the variable in memory: a SLOAD cost 100gas, while MLOAD and MSTORE cost 3 gas.


## PROOF OF CONCEPT

Instances include:

### NestedFactory.sol

scope: `_transferFeeWithRoyalty()`

- `feeSplitter` is read 3 times

[line 573](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/NestedFactory.sol#L573)\
[line 575](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/NestedFactory.sol#L575)\
[line 577](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/NestedFactory.sol#L577)


## TOOLS USED

Manual Analysis

## MITIGATION

cache these storage variables in memory

# Calldata instead of memory for RO function parameters

## PROBLEM

If a reference type function parameter is read-only, it is cheaper in gas to use calldata instead of memory.
Calldata is a non-modifiable, non-persistent area where function arguments are stored, and behaves mostly like memory.

Try to use calldata as a data location because it will avoid copies and also makes sure that the data cannot be modified.

## PROOF OF CONCEPT

Instances include:

### ExchangeHelpers.sol

scope: `fillQuote()`

- [bytes memory _swapCallData](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/libraries/ExchangeHelpers.sol#L18)

### OwnerProxy.sol

scope: `execute()`

- [bytes memory _data](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/governance/OwnerProxy.sol#L16)


## TOOLS USED

Manual Analysis

## MITIGATION

Replace `memory` with `calldata`

# Comparison Operators

## IMPACT

In the EVM, there is no opcode for ` >=` or `<=`.
When using greater than or equal, two operations are performed: `>` and `=`.

Using strict comparison operators hence saves gas

## PROOF OF CONCEPT

Instances include:

### NestedFactory.sol

[_entryFees <= 10000](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/NestedFactory.sol#L161)\
[_entryFees <= 10000](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/NestedFactory.sol#L169)\
[amountSpent <= _inputTokenAmount - feesAmount](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/NestedFactory.sol#L379)\
[amountSpent <= _inputTokenAmount](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/NestedFactory.sol#L428)\
[amounts[1] <= _amountToSpend](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/NestedFactory.sol#L495)\
[address(this).balance >= _inputTokenAmount](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/NestedFactory.sol#L544)\
[nestedRecords.getAssetHolding(_nftId, address(_inputToken)) >= _inputTokenAmount](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/NestedFactory.sol#L552)

### BeefyVaultOperator.sol

[vaultAmount >= minVaultAmount](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/operators/Beefy/BeefyVaultOperator.sol#L54)

### BeefyZapBiswapLPVaultOperator.sol

[vaultAmount >= minVaultAmount](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/operators/Beefy/lp/BeefyZapBiswapLPVaultOperator.sol#L64)\
[amountToDeposit >= depositedAmount](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/operators/Beefy/lp/BeefyZapBiswapLPVaultOperator.sol#L65)\
[tokenAmount >= minTokenAmount](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/operators/Beefy/lp/BeefyZapBiswapLPVaultOperator.sol#L109)

### BeefyZapUniswapLPVaultOperator.sol

[vaultAmount >= minVaultAmount](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/operators/Beefy/lp/BeefyZapUniswapLPVaultOperator.sol#L64)\
[amountToDeposit >= depositedAmount](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/operators/Beefy/lp/BeefyZapUniswapLPVaultOperator.sol#L65)\
[tokenAmount >= minTokenAmount](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/operators/Beefy/lp/BeefyZapUniswapLPVaultOperator.sol#L109)


## TOOLS USED

Manual Analysis

## MITIGATION

Replace `<=` with `<`, and `>=` with `>`. Do not forget to increment/decrement the compared variable

example:

```
-vaultAmount >= minVaultAmount;
+vaultAmount > minVaultAmount - 1;
```

However, if `1` is negligible compared to the value of the variable, we can omit the increment.


# Constructor parameters should be avoided when possible

## IMPACT

Constructor parameters are expensive. The contract deployment will be cheaper in gas if they are hard coded instead of using constructor parameters. With the compilers parameters in `hardhat.config.ts`, deployment costs approximately `400` more gas per variable written via a constructor parameter.

## PROOF OF CONCEPT

Instances include:

### NestedFactory.sol

[constructor](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/NestedFactory.sol#L76-L81)

```
nestedAsset = _nestedAsset;
nestedRecords = _nestedRecords;
reserve = _reserve;
feeSplitter = _feeSplitter;
weth = _weth;
withdrawer = _withdrawer
```

### Withdrawer.sol

[weth = _weth](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/Withdrawer.sol#L17)

### Paraswap.sol

[tokenTransferProxy = _tokenTransferProxy](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/operators/Paraswap/ParaswapOperator.sol#L17)\
[augustusSwapper = _augustusSwapper](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/operators/Paraswap/ParaswapOperator.sol#L18)

### YearnCurveVaultOperator.sol

[eth = _eth](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/operators/Yearn/YearnCurveVaultOperator.sol#L48)\
[withdrawer = _withdrawer](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/operators/Yearn/YearnCurveVaultOperator.sol#L50)

### TimelockControllerEmergency.sol

[_minDelay = minDelay](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/governance/TimelockControllerEmergency.sol#L93)

### OperatorScripts.sol

[nestedFactory = _nestedFactory](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/governance/scripts/OperatorScripts.sol#L21)\
[resolver = _resolver](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/governance/scripts/OperatorScripts.sol#L22)


## TOOLS USED

Manual Analysis, hardhat

## MITIGATION

Hardcode storage variables with their initial value instead of writing it during contract deployment with constructor parameters.


# Default value initialization

## IMPACT

If a variable is not set/initialized, it is assumed to have the default value (0, false, 0x0 etc depending on the data type).
Explicitly initializing it with its default value is an anti-pattern and wastes gas.

## PROOF OF CONCEPT

Instances include:

### NestedFactory.sol

[uint256 i = 0](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/NestedFactory.sol#L124)\
[uint256 i = 0](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/NestedFactory.sol#L136)\
[uint256 i = 0](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/NestedFactory.sol#L196)\
[uint256 i = 0](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/NestedFactory.sol#L256)\
[uint256 i = 0](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/NestedFactory.sol#L315)\
[uint256 i = 0](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/NestedFactory.sol#L333)\
[uint256 i = 0](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/NestedFactory.sol#L369)\
[uint256 i = 0](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/NestedFactory.sol#L412)\
[uint256 i = 0](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/NestedFactory.sol#L651)

### OperatorResolver.sol

[uint256 i = 0](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/OperatorResolver.sol#L40)\
[uint256 i = 0](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/OperatorResolver.sol#L60)\
[uint256 i = 0](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/OperatorResolver.sol#L75)

### MixinOperatorResolver.sol

[uint256 i = 0](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/abstracts/MixinOperatorResolver.sol#L37)\
[uint256 i = 0](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/abstracts/MixinOperatorResolver.sol#L56)

### TimelockControllerEmergency.sol

[uint256 i = 0](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/governance/TimelockControllerEmergency.sol#L84)\
[uint256 i = 0](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/governance/TimelockControllerEmergency.sol#L89)\
[uint256 i = 0](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/governance/TimelockControllerEmergency.sol#L234)\
[uint256 i = 0](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/governance/TimelockControllerEmergency.sol#L324)

## TOOLS USED

Manual Analysis

## MITIGATION

Remove explicit initialization for default values.


# Mathematical optimizations

## PROBLEM

X += Y costs `22` more gas than X = X + Y. This can mean a lot of gas wasted in a function call when the computation is repeated `n` times (loops)

## PROOF OF CONCEPT

Instances include:

### NestedFactory.sol

[amountBought -= amountFees](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/NestedFactory.sol#L266)\
[amountSpent += _submitOrder(address(tokenSold),_batchedOrders.orders[i].token,_nftId,_batchedOrders.orders[i],true // always to the reserve)](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/NestedFactory.sol#L370-L376)\
[ethNeeded += _batchedOrders[i].amount](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/NestedFactory.sol#L653)

## TOOLS USED

Manual Analysis

## MITIGATION

use `X = X + Y` instead of `X += Y` (same with `-`)

# Require instead of AND

## IMPACT

Require statements including conditions with the `&&` operator can be broken down in multiple require statements to save gas.

## PROOF OF CONCEPT

Instances include:

### NestedFactory.sol

[require(address(_nestedAsset) != address(0) &&  address(_nestedRecords) != address(0) &&address(_reserve) != address(0) &&address(_feeSplitter) != address(0) &&address(_weth) != address(0) &&  _operatorResolver != address(0) &&  address(_withdrawer) != address(0),  "NF: INVALID_ADDRESS" )](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/NestedFactory.sol#L66-L74)

### BeefyVaultOperator.sol

[require(vaultAmount != 0 && vaultAmount >= minVaultAmount, "BVO: INVALID_AMOUNT_RECEIVED")](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/operators/Beefy/BeefyVaultOperator.sol#L54)

### BeefyZapBiswapLPVaultOperator.sol

[require(vaultAmount != 0 && vaultAmount >= minVaultAmount, "BLVO: INVALID_AMOUNT_RECEIVED")](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/operators/Beefy/lp/BeefyZapBiswapLPVaultOperator.sol#L64)\
[require(depositedAmount != 0 && amountToDeposit >= depositedAmount, "BLVO: INVALID_AMOUNT_DEPOSITED")](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/operators/Beefy/lp/BeefyZapBiswapLPVaultOperator.sol#L65)

### BeefyZapUniswapLPVaultOperator.sol

[require(vaultAmount != 0 && vaultAmount >= minVaultAmount, "BLVO: INVALID_AMOUNT_RECEIVED")](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/operators/Beefy/lp/BeefyZapUniswapLPVaultOperator.sol#L64)\
[require(depositedAmount != 0 && amountToDeposit >= depositedAmount, "BLVO: INVALID_AMOUNT_DEPOSITED")](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/operators/Beefy/lp/BeefyZapUniswapLPVaultOperator.sol#L65)

### ParaswapOperator.sol

[require(_tokenTransferProxy != address(0) && _augustusSwapper != address(0), "PSO: INVALID_ADDRESS")](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/operators/Paraswap/ParaswapOperator.sol#L16)

## TOOLS USED

Manual Analysis

## MITIGATION

Break down the statements in multiple require statements.

```
-require(_tokenTransferProxy != address(0) && _augustusSwapper != address(0), "PSO: INVALID_ADDRESS");
+require(_tokenTransferProxy != address(0)) 
+require(_augustusSwapper != address(0));
```
You can also improve gas savings by using [custom errors](#custom-errors)


# Revert strings length

## IMPACT

Revert strings cost more gas to deploy if the string is larger than 32 bytes. It costs `9,500` gas upon deployment per string exceeding that 32-byte size.

## PROOF OF CONCEPT

Revert strings exceeding 32 bytes include:

### TimelockControllerEmergency.sol

[TimelockController: length mismatch](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/governance/TimelockControllerEmergency.sol#L229)\
[TimelockController: length mismatch](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/governance/TimelockControllerEmergency.sol#L230)\
[TimelockController: operation already scheduled](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/governance/TimelockControllerEmergency.sol#L243)\
[TimelockController: insufficient delay](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/governance/TimelockControllerEmergency.sol#L244)\
[TimelockController: operation cannot be cancelled](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/governance/TimelockControllerEmergency.sol#L256)\
[TimelockController: length mismatch](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/governance/TimelockControllerEmergency.sol#L319)\
[TimelockController: length mismatch](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/governance/TimelockControllerEmergency.sol#L320)\
[TimelockController: operation is not ready](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/governance/TimelockControllerEmergency.sol#L334)\
[TimelockController: missing dependency](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/governance/TimelockControllerEmergency.sol#L335)\
[TimelockController: operation is not ready](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/governance/TimelockControllerEmergency.sol#L342)\
[TimelockController: underlying transaction reverted](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/governance/TimelockControllerEmergency.sol#L359)\
[TimelockController: caller must be timelock](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/governance/TimelockControllerEmergency.sol#L375)


## TOOLS USED

Manual Analysis

## MITIGATION

Write the error strings so that they do not exceed 32 bytes. For further gas savings, consider also using [custom errors](#custom-errors).


# Shifting cheaper than division

## IMPACT

A division by 2 can be calculated by shifting one to the right. While the DIV opcode uses 5 gas, the SHR opcode only uses 3 gas. Furthermore, Solidity's division operation also includes a division-by-0 prevention which is bypassed using shifting.

## PROOF OF CONCEPT

Instances include:

### BeefyZapBiswapLPVaultOperator.sol

[uint256 halfInvestment = investmentA / 2](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/operators/Beefy/lp/BeefyZapBiswapLPVaultOperator.sol#L275)

### BeefyZapUniswapLPVaultOperator.sol

[uint256 halfInvestment = investmentA / 2](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/operators/Beefy/lp/BeefyZapUniswapLPVaultOperator.sol#L275)

## TOOLS USED

Manual Analysis

## MITIGATION

```
-investmentA / 2;
+investmentA >> 1;
```

# Tight Variable Packing

## PROBLEM

Solidity contracts have contiguous 32 bytes (256 bits) slots used in storage.
By arranging the variables, it is possible to minimize the number of slots used within a contract's storage and therefore reduce deployment costs.

address type variables are each of 20 bytes size (way less than 32 bytes). However, they here take up a whole 32 bytes slot (they are contiguous).

As bool type variables are of size 1 byte, there's a slot here that can get saved by moving one bool closer to an address

## PROOF OF CONCEPT

Instances include:

### OwnableProxyDelegation.sol

```
address private _owner; @audit - slot 1

/// @dev Storage slot with the proxy admin (see TransparentUpgradeableProxy from OZ)
bytes32 internal constant _ADMIN_SLOT = bytes32(uint256(keccak256("eip1967.proxy.admin")) - 1);  @audit - slot 2

/// @dev True if the owner is setted
bool public initialized;  @audit - slot 3
```

## TOOLS USED

Manual Analysis

## MITIGATION

Place `initialized` after `_owner` to save one storage slot

```
address private _owner; @audit - slot 1

/// @dev True if the owner is setted
+bool public initialized; 

/// @dev Storage slot with the proxy admin (see TransparentUpgradeableProxy from OZ)
bytes32 internal constant _ADMIN_SLOT = bytes32(uint256(keccak256("eip1967.proxy.admin")) - 1);  @audit - slot 2
```


# Unchecked arithmetic

## IMPACT

The default "checked" behavior costs more gas when adding/diving/multiplying, because under-the-hood those checks are implemented as a series of opcodes that, prior to performing the actual arithmetic, check for under/overflow and revert if it is detected.

if it can statically be determined there is no possible way for your arithmetic to under/overflow (such as a condition in an if statement), surrounding the arithmetic in an `unchecked` block will save gas. This is particularly true in for loops, as it saves some gas at each iteration.

## PROOF OF CONCEPT

Instances include:

### NestedFactory.sol

[i++](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/NestedFactory.sol#L124)\
[i++](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/NestedFactory.sol#L136)\
[i++](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/NestedFactory.sol#L196)\
[i++](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/NestedFactory.sol#L256)\
[i++](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/NestedFactory.sol#L315)\
[i++](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/NestedFactory.sol#L333)\
[i++](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/NestedFactory.sol#L369)\
[i++](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/NestedFactory.sol#L412)\
[i++](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/NestedFactory.sol#L651)

### OperatorResolver.sol

[i++](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/OperatorResolver.sol#L40)\
[i++](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/OperatorResolver.sol#L60)\
[i++](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/OperatorResolver.sol#L75)

### MixinOperatorResolver.sol

[i++](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/abstracts/MixinOperatorResolver.sol#L37)\
[i++](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/abstracts/MixinOperatorResolver.sol#L56)

### BeefyVaultOperator.sol

[i++](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/operators/Beefy/BeefyVaultOperator.sol#L18)

### BeefyZapBiswapLPVaultOperator.sol

[i++](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/operators/Beefy/lp/BeefyZapBiswapLPVaultOperator.sol#L27)

### BeefyZapUniswapLPVaultOperator.sol

[i++](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/operators/Beefy/lp/BeefyZapUniswapLPVaultOperator.sol#L27)

### YearnCurveVaultOperator.sol

[i++](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/operators/Yearn/YearnCurveVaultOperator.sol#L42)

### CurveHelpers.sol

[i++](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/libraries/CurveHelpers/CurveHelpers.sol#L22)\
[i++](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/libraries/CurveHelpers/CurveHelpers.sol#L42)\
[i++](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/libraries/CurveHelpers/CurveHelpers.sol#L62)\
[i++](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/libraries/CurveHelpers/CurveHelpers.sol#L86)

### OperatorScripts.sol

[i++](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/governance/scripts/OperatorScripts.sol#L67)\
[i++](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/governance/scripts/OperatorScripts.sol#L80)


## TOOLS USED

Manual Analysis

## MITIGATION

Place the arithmetic operations in an `unchecked` block

# Unnecessary computation

## IMPACT

When emitting an event that includes a new and an old value, it is cheaper in gas to avoid caching the old value in memory. Instead, emit the event, then save the new value in storage.

## PROOF OF CONCEPT

Instances include:

### OwnableProxyDelegation.sol

[function _setOwner](https://github.com/code-423n4/2022-06-nested/blob/b4a153c943d54755711a2f7b80cbbf3a5bb49d76/contracts/abstracts/OwnableProxyDelegation.sol#L64-L66)


## TOOLS USED

Manual Analysis

## MITIGATION

Replace

```
address oldOwner = _owner;
_owner = newOwner;
emit OwnershipTransferred(oldOwner, newOwner)
```

with

```
emit OwnershipTransferred(_owner_, newOwner)
_owner = newOwner;
```
