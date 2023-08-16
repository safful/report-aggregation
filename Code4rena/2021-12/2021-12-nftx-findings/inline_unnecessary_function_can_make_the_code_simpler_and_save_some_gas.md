## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Inline unnecessary function can make the code simpler and save some gas](https://github.com/code-423n4/2021-12-nftx-findings/issues/159) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-12-nftx/blob/194073f750b7e2c9a886ece34b6382b4f1355f36/nftx-protocol-v2/contracts/solidity/NFTXLPStaking.sol#L285-L290

```solidity=285
    function _deposit(StakingPool memory pool, uint256 amount) internal {
        require(pool.stakingToken != address(0), "LPStaking: Nonexistent pool");
        IERC20Upgradeable(pool.stakingToken).safeTransferFrom(msg.sender, address(this), amount);
        // Timelock for 2 seconds to prevent flash loans.
        _rewardDistributionTokenAddr(pool).timelockMint(msg.sender, amount, 2);
    }
```

https://github.com/code-423n4/2021-12-nftx/blob/194073f750b7e2c9a886ece34b6382b4f1355f36/nftx-protocol-v2/contracts/solidity/NFTXLPStaking.sol#L124-L130

```solidity=124
    function deposit(uint256 vaultId, uint256 amount) external {
        onlyOwnerIfPaused(10);
        // Check the pool in case its been updated.
        updatePoolForVault(vaultId);
        StakingPool memory pool = vaultStakingInfo[vaultId];
        _deposit(pool, amount);
    }
```


`_deposit()` is unnecessary as it's being used only once. Therefore it can be inlined in `deposit()` to make the code simpler and save gas.

## Recommendation

Change to:

```solidity=124
    function deposit(uint256 vaultId, uint256 amount) external {
        onlyOwnerIfPaused(10);
        // Check the pool in case its been updated.
        updatePoolForVault(vaultId);
        StakingPool memory pool = vaultStakingInfo[vaultId];

        require(pool.stakingToken != address(0), "LPStaking: Nonexistent pool");
        IERC20Upgradeable(pool.stakingToken).safeTransferFrom(msg.sender, address(this), amount);
        // Timelock for 2 seconds to prevent flash loans.
        _rewardDistributionTokenAddr(pool).timelockMint(msg.sender, amount, 2);
    }
```

Other examples include:

-   `NFTXFlashSwipe.sol#flashRedeem()`, `NFTXFlashSwipe.sol#flashMint()` can be inlined in `NFTXFlashSwipe.sol#onFlashLoan()`
-   `UniswapV3SparkleEligibility.sol#isRare()` can be inlined in `UniswapV3SparkleEligibility.sol#_checkIfEligible()`


