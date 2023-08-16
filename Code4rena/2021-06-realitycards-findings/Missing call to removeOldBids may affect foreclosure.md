## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- disagree with severity
- resolved

# [Missing call to removeOldBids may affect foreclosure](https://github.com/code-423n4/2021-06-realitycards-findings/issues/109) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Orderbook.removeBids() as commented “///remove bids in closed markets for a given user ///this can reduce the users bidRate and chance to foreclose”

removeOldBids() is performed currently in Market.newRental() and Treasury.deposit() to  “do some cleaning up, it might help cancel their foreclosure” as commented. However, this is missing in the withdrawDeposit() function where the need is the most because user is removing deposit which may lead to foreclosure and is even commented as being useful on L356.

Impact: If we do not remove closed market bids during withdrawDeposit, the closed market bids still get accounted in user's bidrate in the conditional on L357 and therefore do not prevent the foreclosure in withdrawDeposit that may happen in L357-L367. User may get foreclosed because of mis-accounted closed-market bids in the order book.

## Proof of Concept

https://github.com/code-423n4/2021-06-realitycards/blob/86a816abb058cc0ed9b6f5c4a8ad146f22b8034c/contracts/RCOrderbook.sol#L671-L713

https://github.com/code-423n4/2021-06-realitycards/blob/86a816abb058cc0ed9b6f5c4a8ad146f22b8034c/contracts/RCTreasury.sol#L356

https://github.com/code-423n4/2021-06-realitycards/blob/86a816abb058cc0ed9b6f5c4a8ad146f22b8034c/contracts/RCTreasury.sol#L357-L367

https://github.com/code-423n4/2021-06-realitycards/blob/86a816abb058cc0ed9b6f5c4a8ad146f22b8034c/contracts/RCMarket.sol#L704-L705

https://github.com/code-423n4/2021-06-realitycards/blob/86a816abb058cc0ed9b6f5c4a8ad146f22b8034c/contracts/RCTreasury.sol#L300-L301


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Add call to removeOldBids() on L355 of withdrawDeposit() of Treasury.

