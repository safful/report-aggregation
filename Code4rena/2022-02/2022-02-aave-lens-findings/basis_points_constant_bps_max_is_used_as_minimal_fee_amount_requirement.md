## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Basis points constant BPS_MAX is used as minimal fee amount requirement](https://github.com/code-423n4/2022-02-aave-lens-findings/issues/46) 

# Lines of code

https://github.com/code-423n4/2022-02-aave-lens/blob/main/contracts/core/modules/collect/FeeCollectModule.sol#L72


# Vulnerability details

## Impact

Base fee modules require minimum fixed fee amount to be at least BPS_MAX, which is hard coded to be 10000.

This turns out to be a functionality restricting requirement for some currencies.

For example, WBTC (https://etherscan.io/token/0x2260fac5e5542a773aa44fbcfedf7c193bc2c599, #10 in ERC20 token rankings), has decimals of 8 and current market rate around $40k, i.e. if you want to use any WBTC based collect fee, it has to be at least $4 per collect or fee enabled follow.

Tether and USDC (https://etherscan.io/token/0xdac17f958d2ee523a2206206994597c13d831ec7 and https://etherscan.io/token/0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48, #1 and #3) have decimals of 6, so it is at least $0.01 per collect/follow, which also looks a bit tight for a hard floor minimum.

## Proof of Concept

BPS_MAX is a system wide constant, now 10000:

https://github.com/code-423n4/2022-02-aave-lens/blob/main/contracts/core/modules/FeeModuleBase.sol#L17

https://github.com/code-423n4/2022-02-aave-lens/blob/main/contracts/core/modules/ModuleGlobals.sol#L20

This is correct for any fees defined in basis point terms.

When it comes to the nominal amount, 10000 can be too loose or too tight depending on a currency used, as there can be various combinations of decimals and market rates. 

The following base collect module implementations require fee amount to be at least BPS_MAX (initialization reverts when amount < BPS_MAX):

All collect module implementations use the same check:

FeeCollectModule:

https://github.com/code-423n4/2022-02-aave-lens/blob/main/contracts/core/modules/collect/FeeCollectModule.sol#L72

LimitedFeeCollectModule:

https://github.com/code-423n4/2022-02-aave-lens/blob/main/contracts/core/modules/collect/LimitedFeeCollectModule.sol#L79

TimedFeeCollectModule:

https://github.com/code-423n4/2022-02-aave-lens/blob/main/contracts/core/modules/collect/TimedFeeCollectModule.sol#L81

LimitedTimedFeeCollectModule:

https://github.com/code-423n4/2022-02-aave-lens/blob/main/contracts/core/modules/collect/LimitedTimedFeeCollectModule.sol#L86


FeeFollowModule also uses the same approach:

https://github.com/code-423n4/2022-02-aave-lens/blob/main/contracts/core/modules/follow/FeeFollowModule.sol#L62

## Recommended Mitigation Steps

As a simplest solution consider adding a separate constant for minimum fee amount in nominal terms, say 1 or 10


