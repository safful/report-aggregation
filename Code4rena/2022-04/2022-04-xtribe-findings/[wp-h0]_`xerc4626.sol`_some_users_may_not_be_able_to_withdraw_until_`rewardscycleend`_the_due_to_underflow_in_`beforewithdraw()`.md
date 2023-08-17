## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed
- sponsor todo

# [[WP-H0] `xERC4626.sol` Some users may not be able to withdraw until `rewardsCycleEnd` the due to underflow in `beforeWithdraw()`](https://github.com/code-423n4/2022-04-xtribe-findings/issues/48) 

# Lines of code

https://github.com/fei-protocol/ERC4626/blob/2b2baba0fc480326a89251716f52d2cfa8b09230/src/xERC4626.sol#L65-L68


# Vulnerability details

https://github.com/fei-protocol/ERC4626/blob/2b2baba0fc480326a89251716f52d2cfa8b09230/src/xERC4626.sol#L65-L68

```solidity
function beforeWithdraw(uint256 amount, uint256 shares) internal virtual override {
    super.beforeWithdraw(amount, shares);
    storedTotalAssets -= amount;
}
```

https://github.com/fei-protocol/ERC4626/blob/2b2baba0fc480326a89251716f52d2cfa8b09230/src/xERC4626.sol#L78-L87

```solidity
function syncRewards() public virtual {
    uint192 lastRewardAmount_ = lastRewardAmount;
    uint32 timestamp = block.timestamp.safeCastTo32();

    if (timestamp < rewardsCycleEnd) revert SyncError();

    uint256 storedTotalAssets_ = storedTotalAssets;
    uint256 nextRewards = asset.balanceOf(address(this)) - storedTotalAssets_ - lastRewardAmount_;

    storedTotalAssets = storedTotalAssets_ + lastRewardAmount_; // SSTORE
    ...
```

`storedTotalAssets` is a cached value of total assets which will only include the `unlockedRewards` when the whole cycle ends.

This makes it possible for `storedTotalAssets -= amount` to revert when the withdrawal amount exceeds `storedTotalAssets`, as the withdrawal amount may include part of the `unlockedRewards` in the current cycle.

### PoC

Given:

- rewardsCycleLength = 100 days

1. Alice `deposit()` 100 TRIBE tokens;
2. The owner transferred 100 TRIBE tokens as rewards and called `syncRewards()`;
3. 1 day later, Alice `redeem()` with all shares, the transaction will revert at `xERC4626.beforeWithdraw()`.

Alice's shares worth 101 TRIBE at this moment, but `storedTotalAssets` = 100, making `storedTotalAssets -= amount` reverts due to underflow.

4. Bob `deposit()` 1 TRIBE tokens;
5. Alice `withdraw()` 101 TRIBE tokens, `storedTotalAssets` becomes `0`;
6. Bob can't even withdraw 1 wei of TRIBE token, as `storedTotalAssets` is now `0`.

If there are no new deposits, both Alice and Bob won't be able to withdraw any of their funds until `rewardsCycleEnd`.

### Recommendation

Consider changing to:

```solidity
function beforeWithdraw(uint256 amount, uint256 shares) internal virtual override {
    super.beforeWithdraw(amount, shares);
    uint256 _storedTotalAssets = storedTotalAssets;
    if (amount >= _storedTotalAssets) {
        uint256 _totalAssets = totalAssets();
        // _totalAssets - _storedTotalAssets == unlockedRewards
        lastRewardAmount -= _totalAssets - _storedTotalAssets;
        lastSync = block.timestamp;
        storedTotalAssets = _totalAssets - amount;
    } else {
        storedTotalAssets = _storedTotalAssets - amount;
    }
}
```

