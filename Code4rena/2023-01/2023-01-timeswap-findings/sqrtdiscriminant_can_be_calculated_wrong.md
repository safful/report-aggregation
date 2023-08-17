## Tags

- bug
- 2 (Med Risk)
- satisfactory
- selected for report
- sponsor confirmed
- M-04

# [sqrtDiscriminant can be calculated wrong](https://github.com/code-423n4/2023-01-timeswap-findings/issues/227) 

# Lines of code

https://github.com/code-423n4/2023-01-timeswap/blob/ef4c84fb8535aad8abd6b67cc45d994337ec4514/packages/v2-library/src/Math.sol#L69-L79


# Vulnerability details

## Impact
Due to the wrong calculation of short and long tokens during the `leverage` and `deleverage` process, the users can suffer financial loss while the protocol will lose fees

## Proof of Concept
The protocol uses `leverage` function to deposit short tokens and receive long tokens. On the opposite, `deleverage` function serves for depositing long tokens and receiving short tokens.

[Leverage Function of TimeswapV2Pool contract](https://github.com/code-423n4/2023-01-timeswap/blob/ef4c84fb8535aad8abd6b67cc45d994337ec4514/packages/v2-pool/src/TimeswapV2Pool.sol#L430-L466)
[Deleverage Function of TimeswapV2Pool contract](https://github.com/code-423n4/2023-01-timeswap/blob/ef4c84fb8535aad8abd6b67cc45d994337ec4514/packages/v2-pool/src/TimeswapV2Pool.sol#L377-L418)

Both functions call the PoolLibrary's `leverage` and `deleverage` functions after input sanitization.

[Leverage Function of PoolLibrary contract](https://github.com/code-423n4/2023-01-timeswap/blob/ef4c84fb8535aad8abd6b67cc45d994337ec4514/packages/v2-pool/src/structs/Pool.sol#L552-L655)
[Deleverage Function of PoolLibrary contract](https://github.com/code-423n4/2023-01-timeswap/blob/ef4c84fb8535aad8abd6b67cc45d994337ec4514/packages/v2-pool/src/structs/Pool.sol#L469-L539)


PoolLibrary's `leverage` and `deleverage` functions update the state of the pool first for the fee growth and compute the `long0Amount`, `long1Amount`, and `shortAmount`. It also checks the transaction type according to the passed parameter types as per the `Transaction` contract's enum types below and calls `ConstantProduct`'s appropriate function accordingly;

```solidity
/// @dev The different kind of deleverage transactions.
enum TimeswapV2PoolDeleverage {
    GivenDeltaSqrtInterestRate,
    GivenLong,
    GivenShort,
    GivenSum
}

/// @dev The different kind of leverage transactions.
enum TimeswapV2PoolLeverage {
    GivenDeltaSqrtInterestRate,
    GivenLong,
    GivenShort,
    GivenSum
}
```

If the transaction type is  `GivenSum`, both `leverage` and `deleverage` functions of PoolLibrary call `ConstantProduct.updateGivenSumLong` for the sum amount of the long position in the base denomination to be withdrawn, and the short position to be deposited. 

```solidity

} else if (param.transaction == TimeswapV2PoolDeleverage.GivenSum) {
	(pool.sqrtInterestRate, longAmount, shortAmount, shortFees) = ConstantProduct.updateGivenSumLong(
...
```
[Link](https://github.com/code-423n4/2023-01-timeswap/blob/ef4c84fb8535aad8abd6b67cc45d994337ec4514/packages/v2-pool/src/structs/Pool.sol#L516-L517)

```solidity
} else if (param.transaction == TimeswapV2PoolLeverage.GivenSum) {
	(pool.sqrtInterestRate, longAmount, shortAmount, ) = ConstantProduct.updateGivenSumLong(
```
[Link](https://github.com/code-423n4/2023-01-timeswap/blob/ef4c84fb8535aad8abd6b67cc45d994337ec4514/packages/v2-pool/src/structs/Pool.sol#L602-L603)



 `updateGivenSumLong` updates the new square root interest rate given the sum of long positions in base denomination change and short position change;
 
 
```solidity
    function updateGivenSumLong(
        uint160 liquidity,
        uint160 rate,
        uint256 sumAmount,
        uint96 duration,
        uint256 transactionFee,
        bool isAdd
    ) internal pure returns (uint160 newRate, uint256 longAmount, uint256 shortAmount, uint256 fees) {
        uint256 amount = getShortOrLongFromGivenSum(liquidity, rate, sumAmount, duration, transactionFee, isAdd);

        if (isAdd) (newRate, ) = getNewSqrtInterestRateGivenShort(liquidity, rate, amount, duration, false);
        else newRate = getNewSqrtInterestRateGivenLong(liquidity, rate, amount, false);

        fees = FeeCalculation.getFeesRemoval(amount, transactionFee);
        amount -= fees;

        if (isAdd) {
            shortAmount = amount;
            longAmount = sumAmount - shortAmount;
        } else {
            longAmount = amount;
            shortAmount = sumAmount - longAmount;
        }
    }
```
[Link](https://github.com/code-423n4/2023-01-timeswap/blob/ef4c84fb8535aad8abd6b67cc45d994337ec4514/packages/v2-pool/src/libraries/ConstantProduct.sol#L230-L253)

And `updateGivenSumLong` calls `getShortOrLongFromGivenSum` in order to return the `amount` which represents the short amount or long amount calculated.

```solidity
    function getShortOrLongFromGivenSum(uint160 liquidity, uint160 rate, uint256 sumAmount, uint96 duration, uint256 transactionFee, bool isShort) private pure returns (uint256 amount) {
        uint256 negativeB = calculateNegativeB(liquidity, rate, sumAmount, duration, transactionFee, isShort);
        uint256 sqrtDiscriminant = calculateSqrtDiscriminant(liquidity, rate, sumAmount, duration, transactionFee, negativeB, isShort);
        amount = (negativeB - sqrtDiscriminant).shr(1, false);
    }
```
[Link](https://github.com/code-423n4/2023-01-timeswap/blob/ef4c84fb8535aad8abd6b67cc45d994337ec4514/packages/v2-pool/src/libraries/ConstantProduct.sol#L356-L362)

And the formula needs `sqrtDiscriminant` value to calculate the `amount` and it calls `calculateSqrtDiscriminant` [accordingly](https://github.com/code-423n4/2023-01-timeswap/blob/ef4c84fb8535aad8abd6b67cc45d994337ec4514/packages/v2-pool/src/libraries/ConstantProduct.sol#L395)

`calculateSqrtDiscriminant` function performs a bunch of checks and carries out mathematical functions to return the SqrtDiscriminant by utilizing FullMath and Math libraries.
 
 ```solidity
 sqrtDiscriminant = FullMath.sqrt512(b0, b1, true);
 ```
 [Link](https://github.com/code-423n4/2023-01-timeswap/blob/ef4c84fb8535aad8abd6b67cc45d994337ec4514/packages/v2-pool/src/libraries/ConstantProduct.sol#L430)

The `sqrt` formula in the Math contract uses the modified version of [Babylonian Method](https://en.wikipedia.org/wiki/Methods_of_computing_square_roots#Babylonian_method) when flags are included.

```solidity
    function sqrt(uint256 value, bool roundUp) internal pure returns (uint256 result) { 
        if (value == type(uint256).max) return result = type(uint128).max;
        if (value == 0) return 0;
        unchecked {
            uint256 estimate = (value + 1) >> 1;
            result = value;
            while (estimate < result) {
                result = estimate;
                estimate = (value / estimate + estimate) >> 1;
            }
        }

        if (roundUp && value % result != 0) result++;
    }
```
[Link](https://github.com/code-423n4/2023-01-timeswap/blob/ef4c84fb8535aad8abd6b67cc45d994337ec4514/packages/v2-library/src/Math.sol#L69-L79)

However, when the parameter `roundUp` is passed as `true`, this results in inconsistent behavior for different values. And it's being passed as true as can be seen [here](https://github.com/code-423n4/2023-01-timeswap/blob/ef4c84fb8535aad8abd6b67cc45d994337ec4514/packages/v2-pool/src/libraries/ConstantProduct.sol#L430)).

In order to show some examples let's pass the numbers as values and flag them true by using Math's `sqrt` function.

 Values |1|2|3|4|5|6|7|8|9|10|11|12|13|14|15|16|17|18|19|20|21|22|23|24|25|26|27|28|29|30|
--- | --- | --- | --- |--- |--- |--- |--- |--- |--- |--- |---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---
Results |1|1|1|2|3|**2**|3|**2**|3|4|4|**3**|4|4|**3**|4|5|5|5|**4**|5|5|5|**4**|5|6|6|6|6|**5**| 


 As can be seen from the table, the results are not distributed logically. And many times the result is steeply lesser than its neighbor results. (E.g Sqrt(6) ->2, Sqrt(15)->3 etc.)
 The phenomenon occurs most if the values are small numbers. 

So if the parameter  `value1` in `FullMath.sqrt512` is passed/calculated as zero value, it has a high chance of providing a wrong calculation as a result with the line below;
```solidity
    function sqrt512(uint256 value0, uint256 value1, bool roundUp) internal pure returns (uint256 result) {
        if (value1 == 0) result = value0.sqrt(roundUp);
```
This may lead wrong calculation of the `sqrtDiscriminant`, hence the wrong calculation of short or long amounts for the given transaction. The users might lose financial value due to this. Accordingly, the protocol might lose unspent fees as well.

While the fewer values are affected more on this one, the pools with fewer token decimals and fewer token amounts are more affected by this error. As an example, a [Gemini Dollar](https://coinmarketcap.com/currencies/gemini-dollar/) pool (59th rank on CMC and having 2 decimals) would be subject to false returns.

## Tools Used
Manual Review, Remix, Excel
## Recommended Mitigation Steps
The team might consider not using `true` flag for `Math.sqrt` function.