## Tags

- bug
- 3 (High Risk)
- satisfactory
- selected for report
- sponsor confirmed
- H-01

# [`RewardThrottle.checkRewardUnderflow()` might track the cumulative `APR`s wrongly.](https://github.com/code-423n4/2023-02-malt-findings/issues/23) 

# Lines of code

https://github.com/code-423n4/2023-02-malt/blob/main/contracts/RewardSystem/RewardThrottle.sol#L445-L455
https://github.com/code-423n4/2023-02-malt/blob/main/contracts/RewardSystem/RewardThrottle.sol#L576


# Vulnerability details

## Impact
`RewardThrottle.checkRewardUnderflow()` might calculate the cumulative `APR`s for epochs wrongly.

As a result, `cashflowAverageApr` will be calculated incorrectly in `updateDesiredAPR()`, and `targetAPR` would be changed unexpectedly.

## Proof of Concept
In `checkRewardUnderflow()`, it calls a `_sendToDistributor()` function to update cumulative `APR`s after requesting some capitals from the overflow pool.

```solidity
File: 2023-02-malt\contracts\RewardSystem\RewardThrottle.sol
445:     if (epoch > _activeEpoch) {
446:       for (uint256 i = _activeEpoch; i < epoch; ++i) {
447:         uint256 underflow = _getRewardUnderflow(i);
448: 
449:         if (underflow > 0) {
450:           uint256 balance = overflowPool.requestCapital(underflow);
451: 
452:           _sendToDistributor(balance, i);  //@audit cumulative apr will be tracked wrongly when epoch > _activeEpoch + 1
453:         }
454:       }
455:     }
```

The main reason for this issue is that `_sendToDistributor()` doesn't update the cumulative `APR`s when `amount == 0` and the below scenario would be possible.

1. Let's assume `activeEpoch = 100` and `epoch = 103`. It's possible if the active epoch wasn't updated for 2 epochs.
2. After that, the `checkRewardUnderflow()` function will call `_fillInEpochGaps()` and the cumulative `APR`s will be settled accordingly.
3. And it will try to request capitals from the `overflowPool` and increase the rewards for epochs.
4. At epoch 100, it requests some positive `balance` from `overflowPool` and increases the cumulative `APR`s for epoch 101 correctly in `_sendToDistributor()`.

```solidity
File: 2023-02-malt\contracts\RewardSystem\RewardThrottle.sol
611:     state[epoch].rewarded = state[epoch].rewarded + rewarded;
612:     state[epoch + 1].cumulativeCashflowApr = 
613:       state[epoch].cumulativeCashflowApr +
614:       epochCashflowAPR(epoch);
615:     state[epoch + 1].cumulativeApr = 
616:       state[epoch].cumulativeApr +
617:       epochAPR(epoch);
618:     state[epoch].bondedValue = bonding.averageBondedValue(epoch);
```
5. After that, the `overflowPool` doesn't have any remaining funds and the `balance(At L450)` will be 0 for epochs 101, 102.
6. So `_sendToDistributor()` will be terminated right away and won't increase the cumulative `APR`s of epoch 102 according to epoch 101 and this value won't be changed anymore because the `activeEpoch` is 103 already.

```solidity
File: 2023-02-malt\contracts\RewardSystem\RewardThrottle.sol
575:   function _sendToDistributor(uint256 amount, uint256 epoch) internal { 
576:     if (amount == 0) {
577:       return;
578:     }
```

As a result, the cumulative `APR`s will save smaller values from epoch 102 and `cashflowAverageApr` will be smaller also if the `smoothingPeriod` contains such epochs in `updateDesiredAPR()`.

```solidity
File: 2023-02-malt\contracts\RewardSystem\RewardThrottle.sol
139:     uint256 cashflowAverageApr = averageCashflowAPR(smoothingPeriod);
```

So the `updateDesiredAPR()` function will change the `targetAPR` using the smaller average value and the smoothing logic wouldn't work as expected.

## Tools Used
Manual Review

## Recommended Mitigation Steps
I think `_sendToDistributor()` should update the cumulative `APR`s as well when `amount == 0`.

```solidity
  function _sendToDistributor(uint256 amount, uint256 epoch) internal {
    if (amount == 0) {
        state[epoch + 1].cumulativeCashflowApr = state[epoch].cumulativeCashflowApr + epochCashflowAPR(epoch);
        state[epoch + 1].cumulativeApr = state[epoch].cumulativeApr + epochAPR(epoch);
        state[epoch].bondedValue = bonding.averageBondedValue(epoch);

        return;
    }
```