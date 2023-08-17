## Tags

- bug
- QA (Quality Assurance)
- resolved
- sponsor confirmed
- reviewed

# [QA Report](https://github.com/code-423n4/2022-04-backd-findings/issues/165) 

# Lines of code

https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/contracts/oracles/ChainlinkUsdWrapper.sol#L56


# Vulnerability details

## Impact
The current code returns the following:
```
return (roundId_, (answer_ * _ethPrice()) / 1e8, startedAt_, updatedAt_, answeredInRound_);
```
If we're wrapping an asset that's relatively stable to eth price, the `answer` here might not be updated constantly. By returning the startedAt of the last answer update, it's possible that this answer be considered "stale" from the protocol.

## Recommended Mitigation Steps

It's better to return the new `updatedAt_ ` at the greater of the two:  
* `updatedAt_ ` from eth oracle, 
* `updatedAt_ ` from the asset oracle

This way, if asset/eth is unchanged for a while, but there's a eth price move, we capture the correct `updatedAt` timestamp

