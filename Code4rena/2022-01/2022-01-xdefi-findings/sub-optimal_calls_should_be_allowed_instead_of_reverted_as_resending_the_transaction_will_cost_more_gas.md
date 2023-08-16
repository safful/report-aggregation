## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Sub-optimal calls should be allowed instead of reverted as resending the transaction will cost more gas](https://github.com/code-423n4/2022-01-xdefi-findings/issues/116) 

# Handle

WatchPug


# Vulnerability details

In the current implementation, when `_unlockBatch()` is called with `tokenIds_.length == 1`, the transaction will be reverted with an error `USE_UNLOCK`.

Even though it's sub-optimal to use `relockBatch()` and `unlockBatch()` for only 1 tokenId, reverting and requiring the user to resend the transaction to another method still costs more gas than allowing it.

Therefore, we sugguest not to revert in `_unlockBatch()` when `tokenIds_.length == 1`.

https://github.com/XDeFi-tech/xdefi-distribution/blob/3856a42df295183b40c6eee89307308f196612fe/contracts/XDEFIDistribution.sol#L320-L328

```solidity=320
    function _unlockBatch(address account_, uint256[] memory tokenIds_) internal returns (uint256 amountUnlocked_) {
        uint256 count = tokenIds_.length;
        require(count > uint256(1), "USE_UNLOCK");

        // Handle the unlock for each position and accumulate the unlocked amount.
        for (uint256 i; i < count; ++i) {
            amountUnlocked_ += _unlock(account_, tokenIds_[i]);
        }
    }
```

### Recommendation

Change to:

```solidity
    function _unlockBatch(address account_, uint256[] memory tokenIds_) internal returns (uint256 amountUnlocked_) {
        uint256 count = tokenIds_.length;
        require(count > 0, "NO_TOKEN_IDS");

        // Handle the unlock for each position and accumulate the unlocked amount.
        for (uint256 i; i < count; ++i) {
            amountUnlocked_ += _unlock(account_, tokenIds_[i]);
        }
    }
```

