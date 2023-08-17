## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-03

# [Protocol safeguards for time durations are skewed by a factor of 7. Protocol may potentially lock NFT for period of 7 years.](https://github.com/code-423n4/2022-12-forgeries-findings/issues/273) 

# Lines of code

https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L33


# Vulnerability details

## Description

In VRFNFtRandomDraw.sol initialize(), the MONTH_IN_SECONDS variable is used to validate two values:
- configured time between redraws is under 1 month
- recoverTimelock (when NFT can be returned to owner) is less than 1 year in the future

```
if (_settings.drawBufferTime > MONTH_IN_SECONDS) {
    revert REDRAW_TIMELOCK_NEEDS_TO_BE_LESS_THAN_A_MONTH();
}
...
if (
    _settings.recoverTimelock >
    block.timestamp + (MONTH_IN_SECONDS * 12)
) {
    revert RECOVER_TIMELOCK_NEEDS_TO_BE_LESS_THAN_A_YEAR();
}
```

The issue is that MONTH_IN_SECONDS is calculated incorrectly:
```
/// @dev 60 seconds in a min, 60 mins in an hour
uint256 immutable HOUR_IN_SECONDS = 60 * 60;
/// @dev 24 hours in a day 7 days in a week
uint256 immutable WEEK_IN_SECONDS = (3600 * 24 * 7);
// @dev about 30 days in a month
uint256 immutable MONTH_IN_SECONDS = (3600 * 24 * 7) * 30;
```

MONTH_IN_SECONDS multiplies by 7 incorrectly, as it was copied from WEEK_IN_SECONDS. Therefore, actual seconds calculated is equivalent of 7 months.
Therefore, recoverTimelock can be up to a non-sensible value of 7 years, and re-draws every up to 7 months. 

## Impact

Protocol safeguards for time durations are skewed by a factor of 7. Protocol may potentially lock NFT for period of 7 years.

## Tools Used

Manual audit

## Recommended Mitigation Steps

Fix MONTH_IN_SECONDS calculation:
`uint256 immutable MONTH_IN_SECONDS = (3600 * 24) * 30;`