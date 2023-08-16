## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Unnecessary call to lastApplicableTime() in claimReward()](https://github.com/code-423n4/2021-11-streaming-findings/issues/100) 

# Handle

kenzo


# Vulnerability details

Since `claimReward` can only be called after `endRewardLock`, `lastApplicableTime` will always return `endStream`.

## Impact
Some gas can be saved.

## Proof of Concept
`claimReward` will only run if time > endRewardLock (which is >= endStream):
```
require(block.timestamp > endRewardLock, "lock");
```
https://github.com/code-423n4/2021-11-streaming/blob/main/Streaming/src/Locke.sol#L556
`claimReward` is calling `lastApplicableTime`:
```
lastUpdate = lastApplicableTime();
```
https://github.com/code-423n4/2021-11-streaming/blob/main/Streaming/src/Locke.sol#L567
And this is `lastApplicableTime`:
```
return block.timestamp <= endStream ? uint32(block.timestamp) : endStream;
```
https://github.com/code-423n4/2021-11-streaming/blob/main/Streaming/src/Locke.sol#L340
Therefore, it will always return `endStream`.


## Recommended Mitigation Steps
In `claimReward`, change this line:
```
lastUpdate = lastApplicableTime();
```
https://github.com/code-423n4/2021-11-streaming/blob/main/Streaming/src/Locke.sol#L340
To:
```
lastUpdate = endStream;
```

