## Tags

- bug
- 2 (Med Risk)
- primary issue
- selected for report
- sponsor confirmed
- M-03

# [Rounding error in buyQuote might result in free tokens](https://github.com/code-423n4/2022-12-caviar-findings/issues/243) 

# Lines of code

https://github.com/code-423n4/2022-12-caviar/blob/0212f9dc3b6a418803dbfacda0e340e059b8aae2/src/Pair.sol#L398-L400


# Vulnerability details

In order to guarantee the contract does not become insolvent, incoming assets should be rounded up, while outgoing assets should be rounded down. 

The function `buyQuote()` calculates the amount of base tokens required to buy a given amount of fractional tokens. However, this function rounds down the required amount, which is in favor of the buyer (i.e. he/she has to provide less base tokens for the amount of receiving fractional tokens. 

Depending on the amount of current token reserves and the amount of fractional tokens the user wishes to buy, it might be possible to receive free fractional tokens.

Assume the following reserve state: 

- base token reserve: 0,1 WBTC (=`1e7`)
- fractional token reserve: 10.000.000 (=`1e25`)

The user wishes to buy 0,9 fractional tokens (=`9e17`). Then, the function `buyQuote()` will calculate the amount of base tokens as follows: 

`(9e17 * 1000 * 1e7) / ((1e25 - 9e17) * 997) = 0,903`

As division in Solidity will round down, the amount results in `0` amount of base tokens required (WBTC) to buy 0,9 fractional tokens. 

## Impact

Using the example above, 0,9 fractional tokens is a really small amount (`0,1 BTC / 1e7 = +- $0,00017`). Moreover, if the user keeps repeating this attack, the fractional token reserve becomes smaller, which will result in a buyQuote amount of >1, after which the tokens will not be free anymore. 

Additionally, as the contract incorporates a fee of 30bps, it will likely not be insolvent. The downside would be the LP holder, which will receive a fee of less than 30bps. Hence, the impact is rated as medium.

## Tool Used

Manual Review

## Recommended Mitigation Steps

For incoming assets, it’s recommended to round up the required amount. We could use solmate’s `FixedPointMathLib` library to calculate the quote and round up. This way the required amount will always at least be 1 wei:

```solidity
function buyQuote(uint256 outputAmount) public view returns (uint256) {
  return mulDivUp(outputAmount * 1000, baseTokenReserves(), (fractionalTokenReserves() - outputAmount) * 997);
}
```