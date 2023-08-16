## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [`rewardPerToken()` reverts before start time.](https://github.com/code-423n4/2021-11-streaming-findings/issues/147) 

# Handle

jonah1005


# Vulnerability details

## Impact
`rewardPerToken()` is calculated according to `lastApplicableTime`and `lastUpdate`. [Locke.sol#L343-L353](https://github.com/code-423n4/2021-11-streaming/blob/main/Streaming/src/Locke.sol#L343-L353) Since `lastUpdate` is set to `startTime` before the start time. [Locke.sol#L203-L250](https://github.com/code-423n4/2021-11-streaming/blob/main/Streaming/src/Locke.sol#L203-L250), it reverts before the start time.

`lastApplicableTime()) - lastUpdate` would revert when `lastUpdate` is bigger than `lastApplicableTime()`.

## Proof of Concept
This is the web3.py script:
```python
    stream.functions.stake(deposit_amount).transact()
    stream.functions.rewardPerToken().call()
```
Since `rewardPerToken` returns zero when totalVirtualBalance equals zero, we have to stake a few funds to trigger this bug.

## Tools Used
hardhat
## Recommended Mitigation Steps
Recommend to return zero before startTime.
```solidity
    function rewardPerToken() public view returns (uint256) {
        if (totalVirtualBalance == 0 || lastApplicableTime() < startTime) {
            return cumulativeRewardPerToken;
        } else {
            // ∆time*rewardTokensPerSecond*oneDepositToken / totalVirtualBalance
            return cumulativeRewardPerToken + (
                // NOTE: depositDecimalsOne
                ((uint256(lastApplicableTime()) - lastUpdate) * rewardTokenAmount * depositDecimalsOne/streamDuration) 
                / totalVirtualBalance
            );
        }
    }
```

