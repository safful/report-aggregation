## Tags

- bug
- 2 (Med Risk)
- primary issue
- selected for report
- sponsor confirmed
- M-02

# [Price will not always be 18 decimals, as expected and outlined in the comments](https://github.com/code-423n4/2022-12-caviar-findings/issues/141) 

# Lines of code

https://github.com/code-423n4/2022-12-caviar/blob/0212f9dc3b6a418803dbfacda0e340e059b8aae2/src/Pair.sol#L387-L392


# Vulnerability details

## Impact
The `price()` function is expected to return the price of one fractional tokens, represented in base tokens, to 18 decimals of precision. This is laid out clearly in the comments:

> /// @notice The current price of one fractional token in base tokens with 18 decimals of precision.
/// @dev Calculated by dividing the base token reserves by the fractional token reserves.
/// @return price The price of one fractional token in base tokens * 1e18.

However, the formula incorrectly calculates the price to be represented in whatever number of decimals the base token is in. Since there are many common base tokens (such as USDC) that will have fewer than 18 decimals, this will create a large mismatch between expected prices and the prices that result from the function.

## Proof of Concept

Prices are calculated with the following formula, where `ONE = 1e18`:

```solidity
return (_baseTokenReserves() * ONE) / fractionalTokenReserves();
```

We know that `fractionalTokenReserves` will always be represented in 18 decimals. This means that the `ONE` and the 
`fractionalTokenReserves` will cancel each other out, and we are left with the `baseTokenReserves` number of decimals for the final price.

As an example:
- We have $1000 USDC in reserves, which at 6 decimals is 1e9
- We have 1000 fractional tokens in reserve, which at 18 decimals is 1e21
- The price calculation is `1e9 * 1e18 / 1e21 = 1e6`
- While the value should be 1 token, the 1e6 will be interpreted as just 1/1e12 tokens if we expect the price to be in 1e18

## Tools Used

Manual Review

## Recommended Mitigation Steps

The formula should use the decimals value of the `baseToken` to ensure that the decimals of the resulting price ends up with 18 decimals as expected:

```solidity
return (_baseTokenReserves() * 10 ** (36 - ERC20(baseToken).decimals()) / fractionalTokenReserves();
```

This will multiple `baseTokenReserves` by 1e18, and then additionally by the gap between 1e18 and its own decimals count, which will result in the correct decimals value for the outputted price.