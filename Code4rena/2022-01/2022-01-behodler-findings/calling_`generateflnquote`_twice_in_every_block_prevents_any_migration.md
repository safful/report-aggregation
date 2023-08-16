## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Calling `generateFLNQuote` twice in every block prevents any migration](https://github.com/code-423n4/2022-01-behodler-findings/issues/102) 

# Handle

camden


# Vulnerability details

# Impact and PoC
https://github.com/code-423n4/2022-01-behodler/blob/71d8e0cfd9388f975d6a90dffba9b502b222bdfe/contracts/UniswapHelper.sol#L138
In the Uniswap helper, `generateFLNQuote` is public, so any user can generate the latest quote. If you call this twice in any block, then the two latest flan quotes will have a `blockProduced` value of the current block's number.

These quotes are used in the `_ensurePriceStability` function. The last require statement here is key:
https://github.com/code-423n4/2022-01-behodler/blob/71d8e0cfd9388f975d6a90dffba9b502b222bdfe/contracts/UniswapHelper.sol#L283-L285

This function will revert if this statement is false:
```
localFlanQuotes[0].blockProduced - localFlanQuotes[1].blockProduced > VARS.minQuoteWaitDuration
```
Since `VARS.minQuoteWaitDuration` is a `uint256`, it is at least 0
```
localFlanQuotes[0].blockProduced - localFlanQuotes[1].blockProduced > 0
```
But, as we've shown above, we can create a transaction in every block that will make `localFlanQuotes[0].blockProduced - localFlanQuotes[1].blockProduced == 0`. In any block we can make any call to `_ensurePriceStability` revert.

`_ensurePriceStability` is called in the `ensurePriceStability` modifier:
https://github.com/code-423n4/2022-01-behodler/blob/71d8e0cfd9388f975d6a90dffba9b502b222bdfe/contracts/UniswapHelper.sol#L70

This modifier is used in `stabilizeFlan`: 
https://github.com/code-423n4/2022-01-behodler/blob/71d8e0cfd9388f975d6a90dffba9b502b222bdfe/contracts/UniswapHelper.sol#L162

Lastly, `stabilizeFlan` is used in `migrate` in `Limbo.sol`
https://github.com/code-423n4/2022-01-behodler/blob/71d8e0cfd9388f975d6a90dffba9b502b222bdfe/contracts/Limbo.sol#L234

Therefore, we can grief a migration in any block. In reality, the `minQuoteWaitDuration` would be set to a much higher value than 0, making this even easier to grief for people (you only need to call `generateFLNQuote` every `minQuoteWaitDuration - 1` blocks to be safe).

# Mitigation
Mitigation is to just use a time weighted oracle for uniswap.

