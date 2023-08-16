## Tags

- bug
- sponsor confirmed
- 0 (Non-critical)
- resolved

# [Typos](https://github.com/code-423n4/2021-10-tempus-findings/issues/29) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-10-tempus/blob/63f7639aad08f2bba717830ed81e0649f7fc23ee/contracts/mocks/aave/WadRayMath.sol#L59-L59

```solidity
require(a <= (type(uint256).max - halfWAD) / b, "multiplication oveflow");
```

`oveflow` should be `overflow`.

https://github.com/code-423n4/2021-10-tempus/blob/63f7639aad08f2bba717830ed81e0649f7fc23ee/contracts/amm/VecMath.sol#L10-L10

```solidity
/// @dev Substracting two vectors
```

`Substracting` should be `Subtracting`.

https://github.com/code-423n4/2021-10-tempus/blob/63f7639aad08f2bba717830ed81e0649f7fc23ee/contracts/mocks/lido/StETH.sol#L302-L302

```solidity
* @dev This is used for calaulating tokens from shares and vice versa.
```

`calaulating` should be `calculating`.

https://github.com/code-423n4/2021-10-tempus/blob/63f7639aad08f2bba717830ed81e0649f7fc23ee/contracts/amm/TempusAMM.sol#L779-L779

```solidity
//  - endTime is alawys larger than startTime
```

`alawys` should be `always`.

https://github.com/code-423n4/2021-10-tempus/blob/63f7639aad08f2bba717830ed81e0649f7fc23ee/contracts/TempusController.sol#L47-L47

```solidity
/// @param recipient Address of user that recieved Yield Bearing Tokens
```

`recieved` should be `received`.

https://github.com/code-423n4/2021-10-tempus/blob/63f7639aad08f2bba717830ed81e0649f7fc23ee/contracts/TempusController.sol#L163-L163

```solidity
/// @param recipient Address of user that will recieve yield bearing tokens
```

`recieve` should be `receive`.

