## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Inconsistent definition of when an epoch ends](https://github.com/code-423n4/2021-12-pooltogether-findings/issues/54) 

# Handle

harleythedog


# Vulnerability details

## Impact
The implementation of `_getCurrentEpochId` is:
```
function  _getCurrentEpochId(Promotion  memory  _promotion) internal  view  returns (uint256) {
	return (block.timestamp - _promotion.startTimestamp) / _promotion.epochDuration;
}
```

This means that if exactly `_promotion.epochDuration` seconds have elapsed since the start timestamp, then  the current epoch is 1, and the 0th epoch is completed. However, there are the following lines of code in `_calculateRewardAmount`:

```
function  _calculateRewardAmount(
	address  _user,
	Promotion  memory  _promotion,
	uint256  _epochId
) internal  view  returns (uint256) {
	uint256 _epochDuration = _promotion.epochDuration;
	uint256 _epochStartTimestamp = _promotion.startTimestamp + (_epochDuration * _epochId);
	uint256 _epochEndTimestamp = _epochStartTimestamp + _epochDuration;
	require(block.timestamp > _epochEndTimestamp, "TwabRewards/epoch-not-over");
	...
}
```

If exactly `_promotion.epochDuration` seconds have elapsed since the start timestamp, then this function will revert since the require has a `>` instead of a `>=`.

Thus there are two conflicting definitions of when an epoch ends. In the case of `_getCurrentEpochId`, it is when `_promotion.epochDuration` seconds elapse. In the case of `_calculateRewardAmount`, it is when *more than* `_calculateRewardAmount` seconds elapse. This only makes a difference in one exact second, but it is best to be consistent.

## Proof of Concept
See `_getCurrentEpochId` here: https://github.com/pooltogether/v4-periphery/blob/b520faea26bcf60371012f6cb246aa149abd3c7d/contracts/TwabRewards.sol#L276

See `_calculateRewardAmount` here: https://github.com/pooltogether/v4-periphery/blob/b520faea26bcf60371012f6cb246aa149abd3c7d/contracts/TwabRewards.sol#L289

## Tools Used
Inspection

## Recommended Mitigation Steps
Change 
```
require(block.timestamp > _epochEndTimestamp, "TwabRewards/epoch-not-over");
```
to 
```
require(block.timestamp >= _epochEndTimestamp, "TwabRewards/epoch-not-over");
```
in `_calculateRewardAmount`.

