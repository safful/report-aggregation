## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [RewardThrottle:handleReward gas optimizations](https://github.com/code-423n4/2021-11-malt-findings/issues/144) 

# Handle

GiveMeTestEther


# Vulnerability details

## Impact
Storage variable _activeEpoch is read a lot and can be cached in a local variable (epochTmp, maybe choose a better name =)) to save gas. Also the State struct can be loaded into a State storage (currentState, maybe also choose a better name) variable such that we don't have to access the storage array each time.


In the gas optimized code of "Recommended Mitigation Steps" section, the _activeEpoch only gets read once.
Also note after the first "if" we write epoch to the storage variable "_activeEpoch" but then also write epoch to the local var "epochTmp" so we can use this local var in the whole function.

## Proof of Concept
https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/RewardSystem/RewardThrottle.sol#L63

## Tools Used

## Recommended Mitigation Steps

 function handleReward() public {
    uint256 balance = rewardToken.balanceOf(address(this));

    uint256 epoch = dao.epoch();
    uint256 epochTmp = _activeEpoch;
    State storage currentState;

    checkRewardUnderflow();

    if (epoch > epochTmp) {
      _activeEpoch = epoch;
      epochTmp = epoch;

      currentState = _state[epochTmp];

      currentState.bondedValue = bonding.averageBondedValue(epochTmp); 
      currentState.profit = balance;
      currentState.rewarded = 0;
      currentState.throttle = throttle;
    } else {
      currentState = _state[epochTmp];
      currentState.profit = currentState.profit.add(balance);
      currentState.throttle = throttle; 
    }

    // Fetch targetAPR before we update current epoch state
    uint256 aprTarget = targetAPR();

    // Distribute balance to the correct places
    if (aprTarget > 0 && _epochAprGivenReward(epoch, balance) > aprTarget) {
      uint256 remainder = _getRewardOverflow(balance, aprTarget);
      emit RewardOverflow(epochTmp, remainder);

      if (remainder > 0) {
        rewardToken.safeTransfer(address(overflowPool), remainder);

        if (balance > remainder) {
          _sendToDistributor(balance - remainder, epochTmp);
        }
      }
    } else {
      _sendToDistributor(balance, epochTmp);
    }

    emit HandleReward(epoch, balance);
  }

