## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [[WP-G37] Change `public` constant variables to `private` / `internal` can save gas](https://github.com/code-423n4/2022-01-insure-findings/issues/282) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/PoolTemplate.sol#L146-L146

```solidity
uint256 public constant MAGIC_SCALE_1E6 = 1e6; //internal multiplication scale 1e6 to reduce decimal truncation
```

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/IndexTemplate.sol#L95-L95

```solidity
uint256 public constant MAGIC_SCALE_1E6 = 1e6; //internal multiplication scale 1e6 to reduce decimal truncation
```


https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/CDSTemplate.sol#L55-L55

```solidity
uint256 public constant MAGIC_SCALE_1E6 = 1e6; //internal multiplication scale 1e6 to reduce decimal truncation
```


https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/Vault.sol#L38-L38

```solidity
uint256 public constant MAGIC_SCALE_1E6 = 1e6; //internal multiplication scale 1e6 to reduce decimal truncation
```

For the constants that should not be `public`, changing them to `private` / `internal` can save some gas. To avoid unnecessary getter functions.

