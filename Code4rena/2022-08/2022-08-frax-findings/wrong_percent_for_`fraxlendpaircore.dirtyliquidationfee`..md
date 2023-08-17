## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Wrong percent for `FraxlendPairCore.dirtyLiquidationFee`.](https://github.com/code-423n4/2022-08-frax-findings/issues/238) 

# Lines of code

https://github.com/code-423n4/2022-08-frax/blob/c4189a3a98b38c8c962c5ea72f1a322fbc2ae45f/src/contracts/FraxlendPairCore.sol#L194


# Vulnerability details

## Impact
After confirmed with the sponsor, `dirtyLiquidationFee` is 90% of `cleanLiquidationFee` like the [comment](https://github.com/code-423n4/2022-08-frax/blob/c4189a3a98b38c8c962c5ea72f1a322fbc2ae45f/src/contracts/FraxlendPairCore.sol#L194).

But it uses `9% (9000 / 1e5 = 0.09)` and the fee calculation will be wrong [here](https://github.com/code-423n4/2022-08-frax/blob/c4189a3a98b38c8c962c5ea72f1a322fbc2ae45f/src/contracts/FraxlendPairCore.sol#L988-L990).


## Tools Used
Manual Review


## Recommended Mitigation Steps
We should change `9000` to `90000`.

```
dirtyLiquidationFee = (_liquidationFee * 90000) / LIQ_PRECISION; // 90% of clean fee
```