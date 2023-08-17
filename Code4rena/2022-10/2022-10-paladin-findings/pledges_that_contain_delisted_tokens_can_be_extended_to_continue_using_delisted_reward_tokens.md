## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- sponsor confirmed
- selected for report
- M-04

# [Pledges that contain delisted tokens can be extended to continue using delisted reward tokens](https://github.com/code-423n4/2022-10-paladin-findings/issues/145) 

# Lines of code

https://github.com/code-423n4/2022-10-paladin/blob/d6d0c0e57ad80f15e9691086c9c7270d4ccfe0e6/contracts/WardenPledge.sol#L368-L404


# Vulnerability details

## Impact

Delisted reward tokens can continue to be use by extending current pledges that already use it

## Proof of Concept

    if(pledgeId >= pledgesIndex()) revert Errors.InvalidPledgeID();
    address creator = pledgeOwner[pledgeId];
    if(msg.sender != creator) revert Errors.NotPledgeCreator();


    Pledge storage pledgeParams = pledges[pledgeId];
    if(pledgeParams.closed) revert Errors.PledgeClosed();
    if(pledgeParams.endTimestamp <= block.timestamp) revert Errors.ExpiredPledge();
    if(newEndTimestamp == 0) revert Errors.NullEndTimestamp();
    uint256 oldEndTimestamp = pledgeParams.endTimestamp;
    if(newEndTimestamp != _getRoundedTimestamp(newEndTimestamp) || newEndTimestamp < oldEndTimestamp) revert Errors.InvalidEndTimestamp();


    uint256 addedDuration = newEndTimestamp - oldEndTimestamp;
    if(addedDuration < minDelegationTime) revert Errors.DurationTooShort();
    uint256 totalRewardAmount = (pledgeParams.rewardPerVote * pledgeParams.votesDifference * addedDuration) / UNIT;
    uint256 feeAmount = (totalRewardAmount * protocalFeeRatio) / MAX_PCT ;
    if(totalRewardAmount > maxTotalRewardAmount) revert Errors.IncorrectMaxTotalRewardAmount();
    if(feeAmount > maxFeeAmount) revert Errors.IncorrectMaxFeeAmount();

During the input validation checks, it's never checked that reward token of the pledge being extended is still a valid reward token. This would allow creators using delisted tokens to continue using them as long as they wanted, by simply extending their currently active pledges.

## Tools Used

Manual Review

## Recommended Mitigation Steps

Add the following check during the input validation block:

    +   if(minAmountRewardToken[rewardToken] == 0) revert Errors.TokenNotWhitelisted();