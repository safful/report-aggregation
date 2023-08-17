## Tags

- bug
- 2 (Med Risk)
- satisfactory
- selected for report
- sponsor confirmed
- edited-by-warden
- M-15

# [LinearDistributor.declareReward: previouslyVested may update incorrectly, which will cause some rewards to be lost](https://github.com/code-423n4/2023-02-malt-findings/issues/6) 

# Lines of code

https://github.com/code-423n4/2023-02-malt/blob/700f9b468f9cf8c9c5cffaa1eba1b8dea40503f9/contracts/RewardSystem/LinearDistributor.sol#L111-L153


# Vulnerability details

## Impact
In LinearDistributor.declareReward , distributed represents the reward to distribute and is calculated using netVest(currentlyVested - previouslyVested).
At the same time, distributed cannot exceed balance, which means that if balance < linearBondedValue * netVest / vestingBondedValue, part of the rewards in netVest will be lost.
```solidity
    uint256 netVest = currentlyVested - previouslyVested;
    uint256 netTime = block.timestamp - previouslyVestedTimestamp;

    if (netVest == 0 || vestingBondedValue == 0) {
      return;
    }

    uint256 linearBondedValue = rewardMine.valueOfBonded();

    uint256 distributed = (linearBondedValue * netVest) / vestingBondedValue;
    uint256 balance = collateralToken.balanceOf(address(this));

    if (distributed > balance) {
      distributed = balance;
    }

```
At the end of the function, previouslyVested is directly assigned to currentlyVested instead of using the Vested adjusted according to distributed, which means that the previously lost rewards will also be skipped in the next distribution.
```solidity
    previouslyVested = currentlyVested;
    previouslyVestedTimestamp = block.timestamp;
```
Also, in the next distribution, bufferRequirement will be small because distributed is small, so it may increase the number of forfeits.
```
    if (netTime < buf) {
      bufferRequirement = (distributed * buf * 10000) / netTime / 10000;
    } else {
      bufferRequirement = distributed;
    }

    if (balance > bufferRequirement) {
      // We have more than the buffer required. Forfeit the rest
      uint256 net = balance - bufferRequirement;
      _forfeit(net);
    }
```
## Proof of Concept
https://github.com/code-423n4/2023-02-malt/blob/700f9b468f9cf8c9c5cffaa1eba1b8dea40503f9/contracts/RewardSystem/LinearDistributor.sol#L111-L153
## Tools Used
None
## Recommended Mitigation Steps
Consider adapting previouslyVested based on distributed
```diff
    uint256 linearBondedValue = rewardMine.valueOfBonded();

    uint256 distributed = (linearBondedValue * netVest) / vestingBondedValue;
    uint256 balance = collateralToken.balanceOf(address(this));

    if (distributed > balance) {
      distributed = balance;
+    currentlyVested = distributed * vestingBondedValue / linearBondedValue + previouslyVested;
    }
```