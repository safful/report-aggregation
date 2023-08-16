## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [No bar fees for IndexPools?](https://github.com/code-423n4/2021-09-sushitrident-findings/issues/181) 

# Handle

0xsanson


# Vulnerability details

## Impact
IndexPool doesn't collect fees for `barFeeTo`. Since this Pool contains also a method `updateBarFee()`, probably this is an unintended behavior.
Also without a fee, liquidity providers would probably ditch ConstantProductPool in favor of IndexPool (using the same two tokens with equal weights), since they get all the rewards. This would constitute an issue for the ecosystem.

## Recommended Mitigation Steps
Add a way to send barFees to barFeeTo, same as the other pools.

