## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas Optimization: Use unchecked for safe math](https://github.com/code-423n4/2022-01-insure-findings/issues/317) 

# Handle

gzeon


# Vulnerability details

## Impact
Use unchecked for safe math to save gas, for example:
https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/PremiumModels/BondingPremium.sol#L176
```
        premiumRate = premiumRate / T_1 / (u1 - u2) / BASE;
```
Since we have
1) T_1 != 0 (L229)
2) (u1 - u2) != 0 (L126-132)
3) BASE != 0 (L28)
we can safely wrap this line in an unchecked block

