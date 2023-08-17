## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Wrong reward distribution in Bribe because deliverReward() won't set tokenRewardsPerEpoch[token][epochStart] to 0 ](https://github.com/code-423n4/2022-05-velodrome-findings/issues/141) 

# Lines of code

https://github.com/code-423n4/2022-05-velodrome/blob/7fda97c570b758bbfa7dd6724a336c43d4041740/contracts/contracts/Bribe.sol#L83-L90


# Vulnerability details

## Impact
Function `deliverReward()` in `Bribe` contract won't set `tokenRewardsPerEpoch[token][epochStart]` to `0` after transferring rewards. `Gauge.getReward()` calls `Voter.distribute()` which calls `Gauge.deliverBribes()` which calls `Bribe.deliverReward()`. so if `Gauge.getReward()` or `Voter.distribute()` get called multiple times in same epoch then `deliverReward()` will transfer `Bribe` tokens multiple times because it doesn't set `tokenRewardsPerEpoch[token][epochStart]` to `0` after transferring.

## Proof of Concept
This is `deliverReward()` code in `Bribe`:
```
  function deliverReward(address token, uint epochStart) external lock returns (uint) {
    require(msg.sender == gauge);
    uint rewardPerEpoch = tokenRewardsPerEpoch[token][epochStart];
    if (rewardPerEpoch > 0) {
      _safeTransfer(token, address(gauge), rewardPerEpoch);
    }
    return rewardPerEpoch;
  }
```
As you can see it doesn't set `tokenRewardsPerEpoch[token][epochStart]` value to `0`, so if this function get called multiple times it will transfer epoch rewards multiple times (it will use other epoch's rewards tokens).
function `Gauge.deliverBribes()` calls `Bribe.deliverReward()` and  `Gauge.deliverBribes()` is called by `Voter.distribute()` if the condition `claimable[_gauge] > DURATION` is `True`. This is those functions codes:
```
    function deliverBribes() external lock {
        require(msg.sender == voter);
        IBribe sb = IBribe(bribe);
        uint bribeStart = block.timestamp - (block.timestamp % (7 days)) + BRIBE_LAG;
        uint numRewards = sb.rewardsListLength();

        for (uint i = 0; i < numRewards; i++) {
            address token = sb.rewards(i);
            uint epochRewards = sb.deliverReward(token, bribeStart);
            if (epochRewards > 0) {
                _notifyBribeAmount(token, epochRewards, bribeStart);
            }
        }
    }
```
```
    function distribute(address _gauge) public lock {
        require(isAlive[_gauge]); // killed gauges cannot distribute
        uint dayCalc = block.timestamp % (7 days);
        require((dayCalc < BRIBE_LAG) || (dayCalc > (DURATION + BRIBE_LAG)), "cannot claim during votes period");
        IMinter(minter).update_period();
        _updateFor(_gauge);
        uint _claimable = claimable[_gauge];
        if (_claimable > IGauge(_gauge).left(base) && _claimable / DURATION > 0) {
            claimable[_gauge] = 0;
            IGauge(_gauge).notifyRewardAmount(base, _claimable);
            emit DistributeReward(msg.sender, _gauge, _claimable);
            // distribute bribes & fees too
            IGauge(_gauge).deliverBribes();
        }
    }
```
also `Gauge.getReward()` calls `Voter.getReward()`.
condition `claimable[_gauge] > DURATION` in `Voter.distribute()` can be true multiple time in one epoch (`deliverBribes()` would be called multiple times) because `claimable[_gauge]` is based on `index` and `index` increase by `notifyRewardAmount()` in `Voter` anytime.

## Tools Used
VIM

## Recommended Mitigation Steps
set `tokenRewardsPerEpoch[token][epochStart]` to `0` in `deliverReward`

