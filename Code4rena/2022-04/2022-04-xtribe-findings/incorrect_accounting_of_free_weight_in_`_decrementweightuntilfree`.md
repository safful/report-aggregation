## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- sponsor todo

# [Incorrect accounting of free weight in `_decrementWeightUntilFree`](https://github.com/code-423n4/2022-04-xtribe-findings/issues/61) 

# Lines of code

https://github.com/fei-protocol/flywheel-v2/blob/77bfadf388db25cf5917d39cd9c0ad920f404aad/src/token/ERC20Gauges.sol#L547-L583


# Vulnerability details

## Impact
In `_decrementWeightUntilFree`, the free weight is calculated by `balanceOf[user] - getUserWeight[user]` plus weight freed from non-deprecated gauges. The non-deprecated criteria is unnecessary and lead to incorrect accounting of free weight.

## Proof of Concept
https://github.com/fei-protocol/flywheel-v2/blob/77bfadf388db25cf5917d39cd9c0ad920f404aad/src/token/ERC20Gauges.sol#L547-L583
```
    function _decrementWeightUntilFree(address user, uint256 weight) internal {
        uint256 userFreeWeight = balanceOf[user] - getUserWeight[user];

        // early return if already free
        if (userFreeWeight >= weight) return;

        uint32 currentCycle = _getGaugeCycleEnd();

        // cache totals for batch updates
        uint112 userFreed;
        uint112 totalFreed;

        // Loop through all user gauges, live and deprecated
        address[] memory gaugeList = _userGauges[user].values();

        // Free gauges until through entire list or under weight
        uint256 size = gaugeList.length;
        for (uint256 i = 0; i < size && (userFreeWeight + totalFreed) < weight; ) {
            address gauge = gaugeList[i];
            uint112 userGaugeWeight = getUserGaugeWeight[user][gauge];
            if (userGaugeWeight != 0) {
                // If the gauge is live (not deprecated), include its weight in the total to remove
                if (!_deprecatedGauges.contains(gauge)) {
                    totalFreed += userGaugeWeight;
                }
                userFreed += userGaugeWeight;
                _decrementGaugeWeight(user, gauge, userGaugeWeight, currentCycle);

                unchecked {
                    i++;
                }
            }
        }

        getUserWeight[user] -= userFreed;
        _writeGaugeWeight(_totalWeight, _subtract, totalFreed, currentCycle);
    }
```
Consider Alice allocated 3 weight to gauge D, gauge A and gauge B equally where gauge D is depricated
1. Alice call _decrementWeightUntilFree(alice, 2)
2. userFreeWeight = 0
3. gauge D is freed, totalFreed = 0, userFreed = 1
4. (userFreeWeight + totalFreed) < weight, continue to free next gauge
5. gauge A is freed, totalFreed = 1, userFreed = 2
6. (userFreeWeight + totalFreed) < weight, continue to free next gauge
7. gauge B is freed, totalFreed = 2, userFreed = 3
8. All gauge is freed

Alternatively, Alice can
1. Alice call _decrementWeightUntilFree(alice, 1)
2. userFreeWeight = balanceOf[alice] - getUserWeight[alice] = 3 - 3 = 0
3. gauge D is freed, totalFreed = 0, userFreed = 1
4. (userFreeWeight + totalFreed) < weight, continue to free next gauge
5. gauge A is freed, totalFreed = 1, userFreed = 2
6. (userFreeWeight + totalFreed) >= weight, break
7. getUserWeight[alice] -= totalFreed
8. Alice call _decrementWeightUntilFree(alice, 2)
9. userFreeWeight = balanceOf[alice] - getUserWeight[alice] = 3 - 1 = 2
10. (userFreeWeight + totalFreed) >= weight, break
11. Only 2 gauge is freed

## Recommended Mitigation Steps
No need to treat deprecated gauge seperately

