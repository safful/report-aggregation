## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [`StabilizerNode` Will Mint An Incentive For Triggering An Auction Even If An Auction Exists Already](https://github.com/code-423n4/2021-11-malt-findings/issues/191) 

# Handle

leastwood


# Vulnerability details

## Impact

`_startAuction` utilises the `SwingTrader` contract to purchase Malt. If `SwingTrader` has insufficient capital to return the price of Malt back to its target price, an auction is triggered with the remaining amount. However, no auction is triggered if the current auction exists, but `msg.sender` is still rewarded for their call to `stabilize`.

## Proof of Concept

`_shouldAdjustSupply` initially checks if the current auction is active, however, it does not check if the current auction exists. There is a key distinction between the `auctionActive` and `auctionExists` functions which are not used consistently. Hence, an auction which is inactive but exists would satisfy the edge case and result in `triggerAuction` simply returning.

https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/Auction.sol#L382-L386
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/Auction.sol#L268-L272
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/StabilizerNode.sol#L342-L344
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/Auction.sol#L873-L888

## Tools Used

Manual code review.

## Recommended Mitigation Steps

Consider using `auctionExists` and `auctionActive` consistently in `StabilizerNode` and `Auction` to ensure this edge case cannot be abused.

