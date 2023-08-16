## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [uint256 types can be uint64](https://github.com/code-423n4/2021-12-pooltogether-findings/issues/58) 

# Handle

sirhashalot


# Vulnerability details

## Impact

The `_calculateRewardAmount()` function in TwabRewards.sol uses the uint256 type for the three variables `_epochDuration`, `_epochStartTimestamp`, and `_epochEndTimestamp`. However, there is no need for these variable to be uint256 instead of uint64 because 1. these variables are later cast as uint64 values anyway 2. the block.timestamp value is orders of magnitude less than the uint64 max value. To expand on this second point, if the the block.timestamp values were on the same order of magnitude as the uint64 max value, then the casting of the uint256 timestamp values to uint64 could cause overflow issues because the OpenZeppelin SafeCast library is not used.

The timestamp values could even be of type uint32 (Uniswap v3 does this in places, and the max uint32 timestamp equates to the year 2106), but since the ITicket.sol contract imported by TwabRewards.sol uses uint64, it would be better to use uint64 to maintain consistency.

## Proof of Concept

The uint256 variables that can be uint64 are found in TwabRewards.sol:
https://github.com/pooltogether/v4-periphery/blob/b520faea26bcf60371012f6cb246aa149abd3c7d/contracts/TwabRewards.sol#L294-L296 

## Tools Used

Manual analysis

## Recommended Mitigation Steps

Make these variables uint64 for gas savings and consistency with Iticket.sol timestamps. Remove unnecessary uint64() casting when all variables in the `_calculateRewardAmount()` function consistently use uint64 types.

