## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed
- old-submission-method

# [`fillOrder()` and `exercise()` may lock Ether sent to the contract, forever](https://github.com/code-423n4/2022-06-putty-findings/issues/226) 

# Lines of code

https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L324
https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L338
https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L436


# Vulnerability details

## Impact
`fillOrder()` and `exercise()` have code paths that require Ether to be sent to them (e.g. using WETH as the base asset, or the provision of the exercise price), and therefore those two functions have the `payable` modifier. However, there are code paths within those functions that do not require Ether. Ether passed to the functions, when the non-Ether code paths are taken, is locked in the contract forever, and the sender gets nothing extra in return for it.


## Proof of Concept
Ether can't be pulled from the `order.maker` during the filling of a long order, so `msg.value` shouldn't be provided here:
```solidity
File: contracts/src/PuttyV2.sol   #1

323           if (order.isLong) {
324               ERC20(order.baseAsset).safeTransferFrom(order.maker, msg.sender, order.premium);
325           } else {
```
https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L323-L325


If the `baseAsset` isn't WETH during order fulfillment, `msg.value` is unused:
```solidity
File: contracts/src/PuttyV2.sol   #2

337               } else {
338                   ERC20(order.baseAsset).safeTransferFrom(msg.sender, order.maker, order.premium);
339               }
```
https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L337-L339


Same for the exercise of call options:
```solidity
File: contracts/src/PuttyV2.sol   #3

435               } else {
436                   ERC20(order.baseAsset).safeTransferFrom(msg.sender, address(this), order.strike);
437               }
```
https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L435-L437


## Tools Used
Code inspection

## Recommended Mitigation Steps
Add a `require(0 == msg.value)` for the above three conditions


