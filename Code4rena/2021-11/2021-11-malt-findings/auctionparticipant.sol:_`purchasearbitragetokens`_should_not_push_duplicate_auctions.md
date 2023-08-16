## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [AuctionParticipant.sol: `purchaseArbitrageTokens` should not push duplicate auctions](https://github.com/code-423n4/2021-11-malt-findings/issues/87) 

# Handle

harleythedog


# Vulnerability details

## Impact
In AuctionParticpant.sol, every time `purchaseArbitrageTokens` is called, the current auction is pushed to `auctionIds`. If this function were to be called on the same auction multiple times, then the same auction id would be pushed multiple times into this array, and the `claim` function would have issues with `replenishingIndex`. 

Specifically, even if `replenishingIndex` was incremented once in `claim`, it is still possible that the auction at the next index will never reward any more tokens to the participant, so the contract would need manual intervention to set `replenishingIndex` (due to the if statement on lines 79-82 that does nothing if there is no claimable yield). 

It is likely that `purchaseArbitrageTokens` would be called multiple times on the same auction. In fact, the commented out code for `handleDeficit` (in ImpliedCollateralService.sol) even suggests that the purchases might happen within the same transaction. So this issue will likely be an issue on most auctions and would require manual setting of `replenishingIndex`.

NOTE: This is a separate issue from the one I just submitted previously relating to `replenishingIndex`. The previous issue was related to an edge case where `replenishingIndex` might need to be incremented by one if there are never going to be more claims, while this issue is due to duplicate auction ids.

## Proof of Concept
See code for `purchaseArbitrageTokens` here: https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/AuctionParticipant.sol#L40

Notice that `currentAuction` is always appended to `auctionIds`. 

## Tools Used
Inspection

## Recommended Mitigation Steps
Add a check to the function to `purchaseArbitrageTokens` to ensure that duplicate ids are not added. For example, this can be achieved by changing auctionIds to a mapping instead of an array.

