## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Fee calculation is potentially incorrect](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/129) 

# Handle

itsmeSTYJ


# Vulnerability details

## Impact
More fees are actually charged than intended 

## Mitigation Steps

[Basket.sol line 118](https://github.com/code-423n4/2021-09-defiProtocol/blob/main/contracts/contracts/Basket.sol#L118) 

Assume that license fee is 10% i.e. 1e17 and time diff = half a year. 

When you calculate `feePct`, you expect to get 5e16 since that's 5% and the actual amount of fee to be charged should be totalSupply * feePct (5) / BASE (100) but on line 118, we are actually dividing by BASE - feePct i.e. 95. 

5 / 95 = 0.052 instead of the intended 0.05.

Solution is to replace `BASE - feePct` in the denominator with `BASE`.

