## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [`DropPerSecond` is not updated homogeneously, the rewards emission can be much higher than expected in some cases](https://github.com/code-423n4/2022-03-paladin-findings/issues/44) 

# Lines of code

https://github.com/code-423n4/2022-03-paladin/blob/9c26ec8556298fb1dc3cf71f471aadad3a5c74a0/contracts/HolyPaladinToken.sol#L715-L743


# Vulnerability details

https://github.com/code-423n4/2022-03-paladin/blob/9c26ec8556298fb1dc3cf71f471aadad3a5c74a0/contracts/HolyPaladinToken.sol#L715-L743

```solidity
function _updateDropPerSecond() internal returns (uint256){
    // If no more need for monthly updates => decrease duration is over
    if(block.timestamp > startDropTimestamp + dropDecreaseDuration) {
        // Set the current DropPerSecond as the end value
        // Plus allows to be updated if the end value is later updated
        if(currentDropPerSecond != endDropPerSecond) {
            currentDropPerSecond = endDropPerSecond;
            lastDropUpdate = block.timestamp;
        }

        return endDropPerSecond;
    }

    if(block.timestamp < lastDropUpdate + MONTH) return currentDropPerSecond; // Update it once a month

    uint256 dropDecreasePerMonth = (startDropPerSecond - endDropPerSecond) / (dropDecreaseDuration / MONTH);
    uint256 nbMonthEllapsed = (block.timestamp - lastDropUpdate) / MONTH;

    uint256 dropPerSecondDecrease = dropDecreasePerMonth * nbMonthEllapsed;

    // We calculate the new dropPerSecond value
    // We don't want to go under the endDropPerSecond
    uint256 newDropPerSecond = currentDropPerSecond - dropPerSecondDecrease > endDropPerSecond ? currentDropPerSecond - dropPerSecondDecrease : endDropPerSecond;

    currentDropPerSecond = newDropPerSecond;
    lastDropUpdate = block.timestamp;

    return newDropPerSecond;
}
```

When current time is `lastDropUpdate + (2*MONTH-1)`:

`nbMonthEllapsed` will be round down to `1`, while it's actually 1.99 months passed, but because of precision loss, the smart contract will believe it's only 1 month elapsed, as a result, `DropPerSecond` will only decrease by 1 * `dropDecreasePerMonth`.

In another word, due to the precision loss in calculating the number of months elapsed, for each `_updateDropPerSecond()` there can be a short of up to `1 * dropDecreasePerMonth` for the decrease of emission rate.

At the very edge case, if all the updates happened just like the scenario above. by the end of the `dropDecreaseDuration`, it will drop only `12 * dropDecreasePerMonth` in total, while it's expected to be `24 * dropDecreasePerMonth`.

So only half of `(startDropPerSecond - endDropPerSecond)` is actually decreased. And the last time `updateDropPerSecond` is called, `DropPerSecond` will suddenly drop to `endDropPerSecond`.

### Impact

As the `DropPerSecond` is not updated correctly, in most of the `dropDecreaseDuration`, the actual rewards emission rate is much higher than expected. As a result, the total rewards emission can be much higher than expected.

### Recommendation

Change to:

```solidity
function _updateDropPerSecond() internal returns (uint256){
    // If no more need for monthly updates => decrease duration is over
    if(block.timestamp > startDropTimestamp + dropDecreaseDuration) {
        // Set the current DropPerSecond as the end value
        // Plus allows to be updated if the end value is later updated
        if(currentDropPerSecond != endDropPerSecond) {
            currentDropPerSecond = endDropPerSecond;
            lastDropUpdate = block.timestamp;
        }

        return endDropPerSecond;
    }

    if(block.timestamp < lastDropUpdate + MONTH) return currentDropPerSecond; // Update it once a month

    uint256 dropDecreasePerMonth = (startDropPerSecond - endDropPerSecond) / (dropDecreaseDuration / MONTH);
    uint256 nbMonthEllapsed = UNIT * (block.timestamp - lastDropUpdate) / MONTH;

    uint256 dropPerSecondDecrease = dropDecreasePerMonth * nbMonthEllapsed / UNIT;

    // We calculate the new dropPerSecond value
    // We don't want to go under the endDropPerSecond
    uint256 newDropPerSecond = currentDropPerSecond - dropPerSecondDecrease > endDropPerSecond ? currentDropPerSecond - dropPerSecondDecrease : endDropPerSecond;

    currentDropPerSecond = newDropPerSecond;
    lastDropUpdate = block.timestamp;

    return newDropPerSecond;
}
```

