## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [RubiconRouter _swap does not pass whole amount to RubiconMarket](https://github.com/code-423n4/2022-05-rubicon-findings/issues/104) 

# Lines of code

https://github.com/code-423n4/2022-05-rubicon/blob/main/contracts/RubiconRouter.sol#L229:#L244


# Vulnerability details

When swapping amongst multiple pairs in RubiconRouter's `_swap`, the fee is wrongly accounted for.

## Impact
Not all of the user's funds would be forwarded to RubinconMarket, therefore the user would lose funds.

## Proof of Concept
The `_swap` function is calculating the pay amount to send to RubiconMarket.sellAllAmount [to be](https://github.com/code-423n4/2022-05-rubicon/blob/main/contracts/RubiconRouter.sol#L232):
```
currentAmount.sub(currentAmount.mul(expectedMarketFeeBPS).div(10000)
```
But this would lead to not all of the funds being pulled by RubiconMarket.
I mathematically show this in [this image](https://i.ibb.co/J5678C3/c4amountlost.jpg).
The correct parameter that needs to be sent to sellAllAmount is:
```
currentAmount.sub(currentAmount.mul(expectedMarketFeeBPS).div(10000+expectedMarketFeeBPS)
```
I mathematically prove this in [this image](https://i.ibb.co/xHzYfzF/c4newparam.jpg).

## Recommended Mitigation Steps
Change the parameter to the abovementioned one.

