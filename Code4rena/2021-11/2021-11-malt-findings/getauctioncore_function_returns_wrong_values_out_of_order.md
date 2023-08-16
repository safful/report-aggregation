## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [getAuctionCore function returns wrong values out of order](https://github.com/code-423n4/2021-11-malt-findings/issues/63) 

# Handle

jayjonah8


# Vulnerability details

## Impact
In the AuctionEscapeHatch.sol file both earlyExitReturn() and _calculateMaltRequiredForExit call the getAuctionCore() function which has 10 possible return values most of which are not used.  It gets the wrong value back for the "active"  variable since it's the 10th argument but both functions have it as the 9th return value where "preAuctionReserveRatio" should be because of one missing comma.  This is serious because these both are functions which deal with allowing a user to exit their arbitrage token position early.  This can result in a loss of user funds. 


## Proof of Concept
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/AuctionEscapeHatch.sol#L100
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/AuctionEscapeHatch.sol#L174
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/Auction.sol#L527


## Tools Used
Manual code review 

## Recommended Mitigation Steps
In AuctionEscapeHatch.sol change the following in _calculateMaltRequiredForExit() and earlyExitReturn() functions:

From: 

(,,,,,
     uint256 pegPrice,
     ,
     uint256 auctionEndTime,
     bool active
    ) = auction.getAuctionCore(_auctionId);

To: 

(,,,,,
     uint256 pegPrice,
     ,
     uint256 auctionEndTime,
     ,
     bool active
    ) = auction.getAuctionCore(_auctionId);


