## Tags

- bug
- 3 (High Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [Inefficiency in the Dutch Auction due to lower duration](https://github.com/code-423n4/2022-05-cally-findings/issues/138) 

# Lines of code

https://github.com/code-423n4/2022-05-cally/blob/1849f9ee12434038aa80753266ce6a2f2b082c59/contracts/src/Cally.sol#L406-L423


# Vulnerability details


The vulnerability or bug is in the implementation of the function getDutchAuctionStrike()
The AUCTION_DURATION is defined as 24 hours, and consider that the dutchAuctionReserveStrike (or reserveStrike) will never be set to 0 by user.

Now if a vault is created with startingStrike value of 55 and reserveStrike of 13.5 , the auction price will drop from 55 to 13.5 midway at ~12 hours.
So, after 12 hours from start of auction, the rate will be constant at reserveStrike of 13.5, and remaining time of 12 hours of auction is a waste.
 
Some other examples :
```
startStrike, reserveStrike, time-to-reach-reserveStrike
55 , 13.5  , ~12 hours
55 , 5     , ~16.7 hours
55 , 1.5   , ~20 hours
5  , 1.5   , ~11 hours
```
## Impact
The impact is high wrt Usability, where users have reduced available time to participate in the auction (when price is expected to change).
The vault-Creators or the option-Buyers may or may not be aware of this inefficiency, i.e., how much effective time is available for auction.

## Proof of Concept
Contract : Cally.sol
Function : getDutchAuctionStrike ()

## Recommended Mitigation Steps
The function getDutchAuctionStrike() can be modified such that price drops to the reserveStrike exactly at 24 hours from start of auction.
```
        /*
            delta = max(auctionEnd - currentTimestamp, 0)
            progress = delta / auctionDuration
            auctionStrike = progress^2 * (startingStrike - reserveStrike)             << Changes here
            strike = auctionStrike + reserveStrike                                    << Changes here
        */
        uint256 delta = auctionEndTimestamp > block.timestamp ? auctionEndTimestamp - block.timestamp : 0;
        uint256 progress = (1e18 * delta) / AUCTION_DURATION;
        uint256 auctionStrike = (progress * progress * (startingStrike-reserveStrike)) / (1e18 * 1e18);

        strike = auctionStrike + reserveStrike;
```


