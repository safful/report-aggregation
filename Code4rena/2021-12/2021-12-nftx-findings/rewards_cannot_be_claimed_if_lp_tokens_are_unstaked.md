## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [Rewards Cannot Be Claimed If LP Tokens Are Unstaked](https://github.com/code-423n4/2021-12-nftx-findings/issues/73) 

# Handle

leastwood


# Vulnerability details

## Impact

`TimelockRewardDistributionTokenImpl` calculates the accumulative reward according to the following function:
```
function accumulativeRewardOf(address _owner) public view returns(uint256) {
  return magnifiedRewardPerShare.mul(balanceOf(_owner)).toInt256()
    .add(magnifiedRewardCorrections[_owner]).toUint256Safe() / magnitude;
}
```

The calculation takes into consideration the LP token balance of token holders. Hence, if the token holder has called `emergencyExit` or `withdraw` in `NFTXLPStaking`, the LP tokens are removed from the staking contract without claiming rewards prior to this action. Therefore, in order for users to claim their fair share of rewards they must restake LP tokens and call `claimRewards`. 

Similarly, `_transfer` in `TimelockRewardDistributionTokenImpl` also does not force the `from` account to claim rewards first.

## Proof of Concept

https://github.com/code-423n4/2021-12-nftx/blob/main/nftx-protocol-v2/contracts/solidity/NFTXLPStaking.sol#L195-L198
```
function withdraw(uint256 vaultId, uint256 amount) external {
    StakingPool memory pool = vaultStakingInfo[vaultId];
    _withdraw(pool, amount, msg.sender);
}
```

https://github.com/code-423n4/2021-12-nftx/blob/main/nftx-protocol-v2/contracts/solidity/NFTXLPStaking.sol#L157-L162
```
function emergencyExit(address _stakingToken, address _rewardToken) external {
    StakingPool memory pool = StakingPool(_stakingToken, _rewardToken);
    TimelockRewardDistributionTokenImpl dist = _rewardDistributionTokenAddr(pool);
    require(isContract(address(dist)), "Not a pool");
    _withdraw(pool, dist.balanceOf(msg.sender), msg.sender);
}
```

https://github.com/code-423n4/2021-12-nftx/blob/main/nftx-protocol-v2/contracts/solidity/token/TimelockRewardDistributionTokenImpl.sol#L199-L206
```
function _transfer(address from, address to, uint256 value) internal override {
  require(block.timestamp > timelock[from], "User locked");
  super._transfer(from, to, value);

  int256 _magCorrection = magnifiedRewardPerShare.mul(value).toInt256();
  magnifiedRewardCorrections[from] = magnifiedRewardCorrections[from].add(_magCorrection);
  magnifiedRewardCorrections[to] = magnifiedRewardCorrections[to].sub(_magCorrection);
}
```

## Tools Used

Manual code review.

## Recommended Mitigation Steps

Consider removing functions in `NFTXLPStaking` that do not claim token rewards before unstaking LP tokens or alternatively add code to the affected functions such that rewards are claimed before withdrawing LP tokens.

