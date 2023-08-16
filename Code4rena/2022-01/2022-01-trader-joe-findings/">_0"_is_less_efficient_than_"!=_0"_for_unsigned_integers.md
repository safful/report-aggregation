## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# ["> 0" is less efficient than "!= 0" for unsigned integers](https://github.com/code-423n4/2022-01-trader-joe-findings/issues/240) 

# Handle

WatchPug


# Vulnerability details

It is cheaper to use `!= 0` than `> 0` for uint256.

https://github.com/code-423n4/2022-01-trader-joe/blob/a1579f6453bc4bf9fb0db9c627beaa41135438ed/contracts/RocketJoeStaking.sol#L101-L101

```solidity
if (user.amount > 0) {
```

https://github.com/code-423n4/2022-01-trader-joe/blob/a1579f6453bc4bf9fb0db9c627beaa41135438ed/contracts/RocketJoeFactory.sol#L119-L119

```solidity
_tokenAmount > 0,
```

https://github.com/code-423n4/2022-01-trader-joe/blob/a1579f6453bc4bf9fb0db9c627beaa41135438ed/contracts/LaunchEvent.sol#L338-L338

```solidity
if (rJoeNeeded > 0) {
```

https://github.com/code-423n4/2022-01-trader-joe/blob/a1579f6453bc4bf9fb0db9c627beaa41135438ed/contracts/LaunchEvent.sol#L355-L355

```solidity
require(_amount > 0, "LaunchEvent: invalid withdraw amount");
```

https://github.com/code-423n4/2022-01-trader-joe/blob/a1579f6453bc4bf9fb0db9c627beaa41135438ed/contracts/LaunchEvent.sol#L370-L370

```solidity
if (feeAmount > 0) {
```

https://github.com/code-423n4/2022-01-trader-joe/blob/a1579f6453bc4bf9fb0db9c627beaa41135438ed/contracts/LaunchEvent.sol#L455-L455

```solidity
if (tokenReserve > 0) {
```

https://github.com/code-423n4/2022-01-trader-joe/blob/a1579f6453bc4bf9fb0db9c627beaa41135438ed/contracts/LaunchEvent.sol#L537-L537

```solidity
if (excessToken > 0) {
```

