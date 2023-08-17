## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [instantUnstake fee can be avoided](https://github.com/code-423n4/2022-06-yieldy-findings/issues/9) 

# Lines of code

https://github.com/code-423n4/2022-06-yieldy/blob/524f3b83522125fb7d4677fa7a7e5ba5a2c0fe67/src/contracts/LiquidityReserve.sol#L196


# Vulnerability details

## Impact
Users can utilize the `instantUnstake` function without paying the liquidity provider fee using rounding errors in the fee calculation. This attack only allows for a relatively small amount of tokens to be unstaked in each call, so is likely not feasible on mainnet. However, on low-cost L2s and for tokens with a small decimal precision it is likely a feasible workaround.

## Proof of Concept
The `instantUnstake` fee is handled by sending the user back `amount - fee`. We can work around the fee by unstaking small amounts (`amount < BASIS_POINTS / fee`) in a loop until reaching the desired amount.

## Tools Used
N/A

## Recommended Mitigation Steps
Avoid using subtraction to calculate the fee as this causes the fee to be rounded down rather than the amount. I'd propose calculating amount less fee using a muldiv operation over (1 - fee). In this case, the fee is effectively rounded up instead of down, so it can never be 0 unless fee is 0. Uniswapv2 uses a similar solution for their LP fee: https://github.com/Uniswap/v2-core/blob/8b82b04a0b9e696c0e83f8b2f00e5d7be6888c79/contracts/UniswapV2Pair.sol#L180-L182

It might look like the following:
```
uint256 amountMinusFee = amount * (BASIS_POINTS - fee) / BASIS_POINTS
```

