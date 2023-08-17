## Tags

- bug
- 2 (Med Risk)
- satisfactory
- selected for report
- sponsor confirmed
- M-11

# [RewardThrottle.setTimekeeper: If changing the timekeeper causes the epoch to change, it will mess up the system](https://github.com/code-423n4/2023-02-malt-findings/issues/21) 

# Lines of code

https://github.com/code-423n4/2023-02-malt/blob/700f9b468f9cf8c9c5cffaa1eba1b8dea40503f9/contracts/RewardSystem/RewardThrottle.sol#L690-L696


# Vulnerability details

## Impact
RewardThrottle.setTimekeeper allows POOL_UPDATER_ROLE to update the timekeeper when RewardThrottle is active, 
```solidity
  function setTimekeeper(address _timekeeper)
    external
    onlyRoleMalt(POOL_UPDATER_ROLE, "Must have pool updater privs")
  {
    require(_timekeeper != address(0), "Not address 0");
    timekeeper = ITimekeeper(_timekeeper);
  }
```
if newTimekeeper.epoch changes, it will cause the following
1. The newTimekeeper.epoch increases, and the user can immediately call checkRewardUnderflow to fill the gap epoch, thereby distributing a large amount of rewards.
2. The newTimekeeper.epoch decreases, and the contract will use the state of the previous epoch. Since the state.rewarded has reached the upper limit, this will cause the current epoch to be unable to receive rewards.
## Proof of Concept
https://github.com/code-423n4/2023-02-malt/blob/700f9b468f9cf8c9c5cffaa1eba1b8dea40503f9/contracts/RewardSystem/RewardThrottle.sol#L690-L696
## Tools Used
None
## Recommended Mitigation Steps
Consider only allowing setTimekeeper to be called when RewardThrottle is not active