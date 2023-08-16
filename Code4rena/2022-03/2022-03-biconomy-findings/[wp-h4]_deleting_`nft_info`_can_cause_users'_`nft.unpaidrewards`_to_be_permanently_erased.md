## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [[WP-H4] Deleting `nft Info` can cause users' `nft.unpaidRewards` to be permanently erased](https://github.com/code-423n4/2022-03-biconomy-findings/issues/135) 

# Lines of code

https://github.com/code-423n4/2022-03-biconomy/blob/db8a1fdddd02e8cc209a4c73ffbb3de210e4a81a/contracts/hyphen/LiquidityFarming.sol#L229-L253


# Vulnerability details

https://github.com/code-423n4/2022-03-biconomy/blob/db8a1fdddd02e8cc209a4c73ffbb3de210e4a81a/contracts/hyphen/LiquidityFarming.sol#L229-L253

```solidity
function withdraw(uint256 _nftId, address payable _to) external whenNotPaused nonReentrant {
    address msgSender = _msgSender();
    uint256 nftsStakedLength = nftIdsStaked[msgSender].length;
    uint256 index;
    for (index = 0; index < nftsStakedLength; ++index) {
        if (nftIdsStaked[msgSender][index] == _nftId) {
            break;
        }
    }

    require(index != nftsStakedLength, "ERR__NFT_NOT_STAKED");
    nftIdsStaked[msgSender][index] = nftIdsStaked[msgSender][nftIdsStaked[msgSender].length - 1];
    nftIdsStaked[msgSender].pop();

    _sendRewardsForNft(_nftId, _to);
    delete nftInfo[_nftId];

    (address baseToken, , uint256 amount) = lpToken.tokenMetadata(_nftId);
    amount /= liquidityProviders.BASE_DIVISOR();
    totalSharesStaked[baseToken] -= amount;

    lpToken.safeTransferFrom(address(this), msgSender, _nftId);

    emit LogWithdraw(msgSender, baseToken, _nftId, _to);
}
```


https://github.com/code-423n4/2022-03-biconomy/blob/db8a1fdddd02e8cc209a4c73ffbb3de210e4a81a/contracts/hyphen/LiquidityFarming.sol#L122-L165

```solidity
function _sendRewardsForNft(uint256 _nftId, address payable _to) internal {
    NFTInfo storage nft = nftInfo[_nftId];
    require(nft.isStaked, "ERR__NFT_NOT_STAKED");

    (address baseToken, , uint256 amount) = lpToken.tokenMetadata(_nftId);
    amount /= liquidityProviders.BASE_DIVISOR();

    PoolInfo memory pool = updatePool(baseToken);
    uint256 pending;
    uint256 amountSent;
    if (amount > 0) {
        pending = ((amount * pool.accTokenPerShare) / ACC_TOKEN_PRECISION) - nft.rewardDebt + nft.unpaidRewards;
        if (rewardTokens[baseToken] == NATIVE) {
            uint256 balance = address(this).balance;
            if (pending > balance) {
                unchecked {
                    nft.unpaidRewards = pending - balance;
                }
                (bool success, ) = _to.call{value: balance}("");
                require(success, "ERR__NATIVE_TRANSFER_FAILED");
                amountSent = balance;
            } else {
                nft.unpaidRewards = 0;
                (bool success, ) = _to.call{value: pending}("");
                require(success, "ERR__NATIVE_TRANSFER_FAILED");
                amountSent = pending;
            }
        } else {
            IERC20Upgradeable rewardToken = IERC20Upgradeable(rewardTokens[baseToken]);
            uint256 balance = rewardToken.balanceOf(address(this));
            if (pending > balance) {
                unchecked {
                    nft.unpaidRewards = pending - balance;
                }
                amountSent = _sendErc20AndGetSentAmount(rewardToken, balance, _to);
            } else {
                nft.unpaidRewards = 0;
                amountSent = _sendErc20AndGetSentAmount(rewardToken, pending, _to);
            }
        }
    }
    nft.rewardDebt = (amount * pool.accTokenPerShare) / ACC_TOKEN_PRECISION;
    emit LogOnReward(_msgSender(), baseToken, amountSent, _to);
}
```

When `withdraw()` is called, `_sendRewardsForNft(_nftId, _to)` will be called to send the rewards.

In `_sendRewardsForNft()`, when `address(this).balance` is insufficient at the moment, `nft.unpaidRewards = pending - balance` will be recorded and the user can get it back at the next time.

However, at L244, the whole `nftInfo` is being deleted, so that `nft.unpaidRewards` will also get erased.

There is no way for the user to get back this `unpaidRewards` anymore.

### Recommendation

Consider adding a new parameter named `force` for `withdraw()`, `require(force || unpaidRewards == 0)` before deleting nftInfo.

