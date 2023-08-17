## Tags

- bug
- QA (Quality Assurance)
- sponsor confirmed

# [QA Report](https://github.com/code-423n4/2022-05-opensea-seaport-findings/issues/74) 

# Quality Report

Repo commit referenced: [`49799ce156d979132c9924a739ae45a38b39ecdd`](https://github.com/ProjectOpenSea/seaport/tree/49799ce156d979132c9924a739ae45a38b39ecdd)

## BasicOrderFulfiller.sol

### 1. Adding a new variable to the assembly

For lines [82-86](https://github.com/ProjectOpenSea/seaport/blob/49799ce156d979132c9924a739ae45a38b39ecdd/contracts/lib/BasicOrderFulfiller.sol#L82-L86), we can define a new YUL variable, `basicOrderType` to remove the need to call `calldataload` twice and also adding the `BasicOrder_basicOrderType_cdPtr` twice to the stack, the modified code block  would look like this:

```solidity
let basicOrderType := calldataload(BasicOrder_basicOrderType_cdPtr)

// Mask all but 2 least-significant bits to derive the order type.
orderType := and(basicOrderType, 3)

// Divide basicOrderType by four to derive the route.
route := div(basicOrderType, 4)
```

This would make the code more readable and also should require less gas.

### 2. Missing parameter NatSpec info

Function `_transferERC20AndFinalize` is missing a NatSpec info for `accumulator`. Need to add:

```solidity
* @param accumulator An open-ended array that collects transfers to execute
*                    against a given conduit in a single call.
```

## ConsiderationStructs.sol

Typo in `SpentItem` NatSpec comment `d` is missing from `and` (Line [`68`](https://github.com/ProjectOpenSea/seaport/blob/49799ce156/contracts/lib/ConsiderationStructs.sol#L68)):

```diff
- * @dev A spent item is translated from a utilized offer item an has four
+  * @dev A spent item is translated from a utilized offer item and has four
```

## CriteriaResolution.sol

### 1. Modify For loops to save gas

On line [`56`](https://github.com/ProjectOpenSea/seaport/blob/49799ce156/contracts/lib/CriteriaResolution.sol#L56) and [`166`](https://github.com/ProjectOpenSea/seaport/blob/49799ce156/contracts/lib/CriteriaResolution.sol#L166) the for loops can be modified to:

```solidity
uint256 i = 0;

// Iterate over each criteria resolver.
for (; i < totalCriteriaResolvers;) {
    ...
    ++i;
}

i = 0;

// Iterate over each advanced order.
for (; i < totalAdvancedOrders;) {
    ...
    ++i;
}
```

Note. We don't need to use `unchecked` for `++i` since both for loop blocks are child nodes of a bigger `unchecked` block.

The same modification can be made for the for loops with the `j` counter on line [`184`](https://github.com/ProjectOpenSea/seaport/blob/49799ce156/contracts/lib/CriteriaResolution.sol#L185) and [`199`](https://github.com/ProjectOpenSea/seaport/blob/49799ce156/contracts/lib/CriteriaResolution.sol#L199).

### 2. Redundant array element lookup
One line `177`, the ith element of `advancedOrders` is lookup even though that element is already have been saved in memory as `advancedOrder`

Current (lines [`168-178`](https://github.com/ProjectOpenSea/seaport/blob/49799ce156/contracts/lib/CriteriaResolution.sol#L168-178)):

```solidity
AdvancedOrder memory advancedOrder = advancedOrders[i];

// Skip criteria resolution for order if not fulfilled.
if (advancedOrder.numerator == 0) {
    continue;
}

// Retrieve the parameters for the order.
OrderParameters memory orderParameters = (
    advancedOrders[i].parameters
);
```

So we can reuse `advancedOrder` and the modified block would be:

```solidity
AdvancedOrder memory advancedOrder = advancedOrders[i];

// Skip criteria resolution for order if not fulfilled.
if (advancedOrder.numerator == 0) {
    continue;
}

// Retrieve the parameters for the order.
OrderParameters memory orderParameters = (
    advancedOrder.parameters
);
```

ie:
```diff
- advancedOrders[i].parameters
+ advancedOrder.parameters
```

## Executor.sol

Using the `AccumulatorArmed` defined constants from `ConsiderationConstants.sol`, we can rewrite [Executor.sol:L433](https://github.com/ProjectOpenSea/seaport/blob/49799ce156/contracts/lib/Executor.sol#L433):

```diff
// file: contracts/lib/Executor.sol:L433

- if (accumulator.length != 64) {
+ if (accumulator.length != AccumulatorArmed) {
```

## FulfillmentApplier.sol

### 1. Typo in comment

The `if` block on lines [`180-183`](https://github.com/ProjectOpenSea/seaport/blob/49799ce156/contracts/lib/FulfillmentApplier.sol#L180-L183) has typos for the comment before the `if` block:

```solidity
// Set the offerer as the receipient if execution amount is nonzero.
if (execution.item.amount == 0) {
    execution.item.recipient = payable(execution.offerer);
}
```

`nonzero` needs to be changed to `zero` and `receipient` to `recipient`

```diff
- // Set the offerer as the receipient if execution amount is nonzero.
+ // Set the offerer as the recipient if execution amount is zero.
```

## GettersAndDerivers.sol

### 1. Typo
Typo in comment on line [`131`](https://github.com/ProjectOpenSea/seaport/blob/49799ce156/contracts/lib/GettersAndDerivers.sol#L131), `offer items` should be `consideration items`:

```diff
- // Iterate over the offer items (not including tips).
+ // Iterate over the consideration items (not including tips).
```

### 2. Unpredictable Memory Location

On line [`170`](https://github.com/ProjectOpenSea/seaport/blob/49799ce156/contracts/lib/GettersAndDerivers.sol#L170) a memory location has been set with an unpredictable address/content:

```solidity
let typeHashPtr := sub(orderParameters, OneWord)
```


## OrderCombiner.sol

For the function `_validateOrdersAndPrepareToFulfill` is the `maximumFulfilled` has been reached, the loop keeps continuing without essentailly doing anything other than wasting gas, since on line `196` it keeps jumping to the next `i` value. The `continue` statement on line `196` should be replaced by the `break` statement to exit the loop. ([Ref](https://github.com/ProjectOpenSea/seaport/blob/49799ce156/contracts/lib/OrderCombiner.sol#L181-L197))

```diff
for (uint256 i = 0; i < totalOrders; ++i) {
    // Retrieve the current order.
    AdvancedOrder memory advancedOrder = advancedOrders[i];

    // Determine if max number orders have already been fulfilled.
    if (maximumFulfilled == 0) {
        // Mark fill fraction as zero as the order will not be used.
        advancedOrder.numerator = 0;

        // Update the length of the orderHashes array.
        assembly {
            mstore(orderHashes, add(i, 1))
        }

        // Continue iterating through the remaining orders.
-        continue;
+        break;
    }
```

## OrderFulfiller.sol

To optimize the for loop in Lines [468-475](https://github.com/ProjectOpenSea/seaport/blob/49799ce156/contracts/lib/OrderFulfiller.sol#L468-L475)

The code block below: 

```solidity
// Skip overflow check as the index for the loop starts at zero.
unchecked {
    // Iterate over the given orders.
    for (uint256 i = 0; i < totalOrders; ++i) {
        // Convert to partial order (1/1 or full fill) and update array.
        advancedOrders[i] = _convertOrderToAdvanced(orders[i]);
    }
}
```

can be changed too:

```solidity
// Skip overflow check as the index for the loop starts at zero.
unchecked {
    // Iterate over the given orders.
    uint256 i = 0;
    for (; i < totalOrders;) {
        // Convert to partial order (1/1 or full fill) and update array.
        advancedOrders[i] = _convertOrderToAdvanced(orders[i]);

        ++i;
    }
}
```

## TokenTransferrer.sol

In the function  `_performERC1155BatchTransfers(ConduitBatch1155Transfer[] calldata batchTransfers)` on line `542` the `data` length offset value is incorrect.

```solidity
// Set the length of the data array in memory to zero.
mstore(
    add(
        BatchTransfer1155Params_data_length_basePtr,
        idsAndAmountsSize
    ),
    0
)
```

This value is used to call a `safeBatchTransferFrom(address from, address to, uint256[] ids, uint256[] amounts, bytes data)` endpoint on a `ERC1155` contract.

The formula for the offset should be:

$$ 

\text{data\_length\_offset} = \mathtt{0x20} + \mathtt{0x04} + \mathtt{0xa0} + \text{idsAndAmountsSize} 
\\ = \mathtt{0xc4} + \text{idsAndAmountsSize}

$$

Currently the offset is $\mathtt{0x104} + \text{idsAndAmountsSize}$ which expands memory 2 words extra more than the necessary amount.

Here is an example (cast from foundry):

```
>> cast abi-encode "safeBatchTransferFrom(address,address,uint256[],uint256[],bytes)" 0x8496a8EDc4A0894123062194015c0Cf86b7A277c 0x8496a8EDc4A0894123062194015c0Cf86b7A277c [] [] 0x

0x00 0000000000000000000000008496a8edc4a0894123062194015c0cf86b7a277c from
0x20 0000000000000000000000008496a8edc4a0894123062194015c0cf86b7a277c to
0x40 00000000000000000000000000000000000000000000000000000000000000a0 head(ids)
0x60 00000000000000000000000000000000000000000000000000000000000000c0 head(amounts)
0x80 00000000000000000000000000000000000000000000000000000000000000e0 head(data)
0xa0 0000000000000000000000000000000000000000000000000000000000000000 ids_length
0xc0 0000000000000000000000000000000000000000000000000000000000000000 amounts_length
0xe0 0000000000000000000000000000000000000000000000000000000000000000 data_length
```

The above ABI encoding does not include the `ZeroSlot` in the memory and also the function signature after adding those according to the instruction in `_performERC1155BatchTransfers` we would get:

```
0x000 0000000000000000000000000000000000000000000000000000000000000000 ZeroSlot
0x004 2eb2c2d6
0x024 0000000000000000000000008496a8edc4a0894123062194015c0cf86b7a277c from
0x044 0000000000000000000000008496a8edc4a0894123062194015c0cf86b7a277c to
0x064 00000000000000000000000000000000000000000000000000000000000000a0 head(ids)
0x084 00000000000000000000000000000000000000000000000000000000000000c0 head(amounts)
0x0a4 00000000000000000000000000000000000000000000000000000000000000e0 head(data)
0x0c4 0000000000000000000000000000000000000000000000000000000000000000 ids_length
0x0e4 0000000000000000000000000000000000000000000000000000000000000000 amounts_length
0x104 0000000000000000000000000000000000000000000000000000000000000000 data_length
```

In this example we have `idsAndAmountsSize` = `0x40` = `TwoWords` and the constant `BatchTransfer1155Params_data_length_basePtr` = `0x104` (based on TokenTransferrerConstants.sol), so according to the line `TokenTransferrer.sol:542` the `data_length` memory offset would need to be `0x144` which is outside of the neccesssary memory area.

To fix this error, these changes need to be applied:

Block before change:

```solidity
// file: contract/lib/TokenTransferrer:L541

// Set the length of the data array in memory to zero.
mstore(
    add(
        BatchTransfer1155Params_data_length_basePtr,
        idsAndAmountsSize
    ),
    0
)

// Determine the total calldata size for the call to transfer.
let transferDataSize := add(
    BatchTransfer1155Params_data_length_basePtr,
    mul(idsLength, TwoWords)
)
```

Block after change:

```solidity
// file: contract/lib/TokenTransferrer:L541

// Determine the total calldata size for the call to transfer.
let transferDataSize := add(
    BatchTransfer1155Params_data_length_basePtr,
    idsAndAmountsSize
)

// Set the length of the data array in memory to zero.
mstore(
    transferDataSize,
    0
)
```

and also the following constant would need to be updated:

```diff
// file: contract/lib/TokenTransferrerConstants.sol:L146

- uint256 constant BatchTransfer1155Params_data_length_basePtr = 0x104;
+ uint256 constant BatchTransfer1155Params_data_length_basePtr = 0xc4;
```

The change above removes uses less arithmetic operation which in turn would also save some gas.

## ZoneInteractions.sol

For the function `_assertRestrictedAdvancedOrderValidity` on lines [104-113](https://github.com/ProjectOpenSea/seaport/blob/49799ce156/contracts/lib/ZoneInteraction.sol#L104-L113), `offerer` and `zone` are passed as two extra inputs although `advancedOrder` contains those 2 parameters.

```solidity
function _assertRestrictedAdvancedOrderValidity(
    AdvancedOrder memory advancedOrder,
    CriteriaResolver[] memory criteriaResolvers,
    bytes32[] memory priorOrderHashes,
    bytes32 orderHash,
    bytes32 zoneHash,
    OrderType orderType,
    address offerer,
    address zone
) internal view {
```

These parameters can be accessed as:

```solidity
advancedOrder.parameters.offerer
advancedOrder.parameters.zone
```

This brings up the question if the extra inputs were provided like that to save gas, since using `advancedOrder.parameters.VARIABLE` would require the compiler to use extra instruction to calculate the memory offset and also load those variables to memory. This extra work was perhaps done in a function calling `_assertRestrictedAdvancedOrderValidity`.

## ZoneInteractionErrors.sol

There is typo on line [`13`](https://github.com/ProjectOpenSea/seaport/blob/49799ce156/contracts/interfaces/ZoneInteractionErrors.sol#L13) in the NatSpec comment:

`offerrer` replaced by `offerer`
```diff
- either the offerrer or the order's zone or approved as valid by the
+ either the offerer or the order's zone or approved as valid by the
```

## For loops
The for loops in these FILES:LINES 

- [Conduit.sol:66](https://github.com/ProjectOpenSea/seaport/blob/49799ce156/contracts/conduit/Conduit.sol#L66)
- [Conduit.sol:130](https://github.com/ProjectOpenSea/seaport/blob/49799ce156/contracts/conduit/Conduit.sol#L130)
- [BasicFulfiller.sol:948](https://github.com/ProjectOpenSea/seaport/blob/49799ce156/contracts/lib/BasicOrderFulfiller.sol#L948)
- [BasicFulFiller.sol:1040](https://github.com/ProjectOpenSea/seaport/blob/49799ce156/contracts/lib/BasicOrderFulfiller.sol#L1040)
- [CriteriaResolution.sol:56](https://github.com/ProjectOpenSea/seaport/blob/49799ce156/contracts/lib/CriteriaResolution.sol#L56)
- [CriteriaResolution.sol:166](https://github.com/ProjectOpenSea/seaport/blob/49799ce156/contracts/lib/CriteriaResolution.sol#L166)
- [CriteriaResolution.sol:184](https://github.com/ProjectOpenSea/seaport/blob/49799ce156/contracts/lib/CriteriaResolution.sol#L184)
- [CriteriaResolution.sol:199](https://github.com/ProjectOpenSea/seaport/blob/49799ce156/contracts/lib/CriteriaResolution.sol#L199)
- [OrderCombiner.sol:181](https://github.com/ProjectOpenSea/seaport/blob/49799ce156/contracts/lib/OrderCombiner.sol#L181)
- [OrderCombiner.sol:247](https://github.com/ProjectOpenSea/seaport/blob/49799ce156/contracts/lib/OrderCombiner.sol#L247)
- [OrderCombiner.sol:291](https://github.com/ProjectOpenSea/seaport/blob/49799ce156/contracts/lib/OrderCombiner.sol#L291)
- [OrderCombiner.sol:373](https://github.com/ProjectOpenSea/seaport/blob/49799ce156/contracts/lib/OrderCombiner.sol#L373)
- [OrderCombiner.sol:473](https://github.com/ProjectOpenSea/seaport/blob/49799ce156/contracts/lib/OrderCombiner.sol#L473)
- [OrderCombiner.sol:498](https://github.com/ProjectOpenSea/seaport/blob/49799ce156/contracts/lib/OrderCombiner.sol#L498)
- [OrderCombiner.sol:577](https://github.com/ProjectOpenSea/seaport/blob/49799ce156/contracts/lib/OrderCombiner.sol#L577)
- [OrderCombiner.sol:598](https://github.com/ProjectOpenSea/seaport/blob/49799ce156/contracts/lib/OrderCombiner.sol#L598)
- [OrderCombiner.sol:621](https://github.com/ProjectOpenSea/seaport/blob/49799ce156/contracts/lib/OrderCombiner.sol#L621)
- [OrderCombiner.sol:754](https://github.com/ProjectOpenSea/seaport/blob/49799ce156/contracts/lib/OrderCombiner.sol#L754)
- [OrderFulfiller.sol:217](https://github.com/ProjectOpenSea/seaport/blob/49799ce156/contracts/lib/OrderFulfiller.sol#L217)
- [OrderFulfiller.sol:306](https://github.com/ProjectOpenSea/seaport/blob/49799ce156/contracts/lib/OrderFulfiller.sol#L306)
- [OrderFulfiller.sol:471](https://github.com/ProjectOpenSea/seaport/blob/49799ce156/contracts/lib/OrderFulfiller.sol#L471)
- [OrderValidator.sol:272](https://github.com/ProjectOpenSea/seaport/blob/49799ce156/contracts/lib/OrderValidator.sol#L272)
- [OrderValidator.sol:350](https://github.com/ProjectOpenSea/seaport/blob/49799ce156/contracts/lib/OrderValidator.sol#L350)

can be changed.

General structure used:
```solidity
for (uint256 i = 0; CONDITION; ++i) { ... }
```

Change to:
```solidity
uint256 i = 0;

for (; CONDITION;) { 
    ...
    unchecked {
         ++i;
    } 
}
```

The `unchecked` inner block might not be necessary since some of the loops are inside a bigger `unchecked` block.

and for the assembly blocks in FILES:LINES:

- [BasicOrderFulfiller.sol:506](https://github.com/ProjectOpenSea/seaport/blob/49799ce156/contracts/lib/BasicOrderFulfiller.sol#L506)
- [BasicOrderFulfiller.sol:615](https://github.com/ProjectOpenSea/seaport/blob/49799ce156/contracts/lib/BasicOrderFulfiller.sol#L615)
- [CriteriaResolution.sol:256](https://github.com/ProjectOpenSea/seaport/blob/49799ce156/contracts/lib/CriteriaResolution.sol#L256)
- [GettersAndDerivers.sol:78](https://github.com/ProjectOpenSea/seaport/blob/49799ce156/contracts/lib/GettersAndDerivers.sol#L78)
- [GettersAndDerivers.sol:133](https://github.com/ProjectOpenSea/seaport/blob/49799ce156/contracts/lib/GettersAndDerivers.sol#L133)

General structure used:
```solidity
for { initial }{ condition }{ increment } { ... }
```

Change to
```solidity
initial

for {}{ condition }{} { 
    ... 
    increment
}
```