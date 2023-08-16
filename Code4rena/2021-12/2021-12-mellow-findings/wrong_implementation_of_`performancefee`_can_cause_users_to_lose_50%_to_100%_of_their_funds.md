## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Wrong implementation of `performanceFee` can cause users to lose 50% to 100% of their funds](https://github.com/code-423n4/2021-12-mellow-findings/issues/91) 

# Handle

WatchPug


# Vulnerability details

A certain amount of lp tokens (shares of the vault) will be minted to the `strategyPerformanceTreasury` as `performanceFee`, the amount is calculated based on the `minLpPriceFactor`.

However, the current formula for `toMint` is wrong, which issues more than 100% of the current totalSupply of the lp token to the `strategyPerformanceTreasury` each time. Causing users to lose 50% to 100% of their funds after a few times.

https://github.com/code-423n4/2021-12-mellow/blob/6679e2dd118b33481ee81ad013ece4ea723327b5/mellow-vaults/contracts/LpIssuer.sol#L269-L271

```solidity=269
address treasury = strategyParams.strategyPerformanceTreasury;
uint256 toMint = (baseSupply * minLpPriceFactor) / CommonLibrary.DENOMINATOR;
_mint(treasury, toMint);
```

### PoC

Given:

- `strategyParams.performanceFee`: `10e7` (1%)

1. Alice deposited `1,000 USDC`, received `1000` lpToken; the totalSupply of the lpToken is now: `1000`;
2. 3 days later, `baseTvl` increased to `1,001 USDC`, Bob deposited `1 USDC` and trigegred `_chargeFees()`:

- Expected Result: `strategyPerformanceTreasury` to receive about `0.01` lpToken (1% of 1 USDC);
- Actual Result: `minLpPriceFactor` is about `1.001`, and `strategyPerformanceTreasury` will received `1001` lpToken as performanceFee; Alice lose 50% of deposited funds.

### Recommendation

Change to:

```solidity
address treasury = strategyParams.strategyPerformanceTreasury;
uint256 toMint = (baseSupply * (minLpPriceFactor - CommonLibrary.DENOMINATOR) * performanceFee  / CommonLibrary.DENOMINATOR) / CommonLibrary.DENOMINATOR;
_mint(treasury, toMint);
```

