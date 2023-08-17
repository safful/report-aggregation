## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [`VE3DLocker.sol` Wrong implementation of inversely traverse for loops always reverts](https://github.com/code-423n4/2022-05-vetoken-findings/issues/150) 

# Lines of code

https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/contracts/VE3DLocker.sol#L305-L329
https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/contracts/VE3DLocker.sol#L349-L373
https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/contracts/VE3DLocker.sol#L376-L396
https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/contracts/VE3DLocker.sol#L399-L415


# Vulnerability details

```solidity
function totalSupplyAtEpoch(uint256 _epoch) external view returns (uint256 supply) {
    uint256 epochStart = uint256(epochs[_epoch].date).div(rewardsDuration).mul(
        rewardsDuration
    );
    uint256 cutoffEpoch = epochStart.sub(lockDuration);

    //traverse inversely to make more current queries more gas efficient
    for (uint256 i = _epoch; i + 1 != 0; i--) {
        Epoch storage e = epochs[i];
        if (uint256(e.date) <= cutoffEpoch) {
            break;
        }
        supply = supply.add(epochs[i].supply);
    }

    return supply;
}
````

In `VE3DLocker.sol`, there are multiple instances in which an inversely traverse for loop is used "to make more current queries more gas efficient".

For example:

- `totalSupplyAtEpoch()`
- `balanceAtEpochOf()`
- `pendingLockAtEpochOf()`
- `totalSupply()`

The implementation of the inversely traverse for loop is inherited from Convex's original version: https://github.com/convex-eth/platform/blob/main/contracts/contracts/CvxLockerV2.sol#L333-L334

However, Convex's locker contract is using Solidity 0.6.12, in which the arithmetic operations will overflow/underflow without revert.

As the solidity version used in the current implementation of `VE3DLocker.sol` is `0.8.7`, and there are some breaking changes in Solidity v0.8.0, including:

> Arithmetic operations revert on underflow and overflow. 

Ref: https://docs.soliditylang.org/en/v0.8.7/080-breaking-changes.html#silent-changes-of-the-semantics

Which makes the current implementation of inversely traverse for loops always reverts.

More specifically:

1. `for (uint i = locks.length - 1; i + 1 != 0; i--) {` will revert when `locks.length == 0` at `locks.length - 1` due to underflow;
2. `for (uint256 i = _epoch; i + 1 != 0; i--) {` will loop until `i == 0` and reverts at `i--` due to underflow.

As a result, all these functions will be malfunctioning and all the internal and external usage of these function will always revert.

### Recommendation

Change `VE3DLocker.sol#L315` to:

```solidity
for (uint256 i = locks.length; i > 0; i--) {
    uint256 lockEpoch = uint256(locks[i - 1].unlockTime).sub(lockDuration);
    //lock epoch must be less or equal to the epoch we're basing from.
    if (lockEpoch <= epochTime) {
        if (lockEpoch > cutoffEpoch) {
            amount = amount.add(locks[i - 1].amount);
```

Change `VE3DLocker.sol#L360` to:

```solidity
for (uint256 i = locks.length; i > 0; i--) {
    uint256 lockEpoch = uint256(locks[i - 1].unlockTime).sub(lockDuration);

    //return the next epoch balance
    if (lockEpoch == nextEpoch) {
        return locks[i - 1].amount;
    } else if (lockEpoch < nextEpoch) {
        //no need to check anymore
        break;
    }
```

Change `VE3DLocker.sol#L387` to:

```solidity
for (uint256 i = epochindex; i > 0; i--) {
    Epoch storage e = epochs[i - 1];
```

Change `VE3DLocker.sol#L406` to:

```solidity
for (uint256 i = _epoch + 1; i > 0; i--) {
    Epoch storage e = epochs[i - 1];
    if (uint256(e.date) <= cutoffEpoch) {
        break;
    }
    supply = supply.add(e.supply);
}
```

