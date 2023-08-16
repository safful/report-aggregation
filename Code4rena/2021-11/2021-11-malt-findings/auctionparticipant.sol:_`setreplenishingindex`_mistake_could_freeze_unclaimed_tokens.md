## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [AuctionParticipant.sol: `setReplenishingIndex` mistake could freeze unclaimed tokens](https://github.com/code-423n4/2021-11-malt-findings/issues/88) 

# Handle

harleythedog


# Vulnerability details

## Impact
In AuctionParticipant.sol, the function `setReplenishingIndex` is an admin function that allows manually setting `replenishingIndex`. As I have shown in my two previous findings, I believe that this function could be called frequently. In my opinion (and Murphy's law would agree), this implies that eventually an admin will accidentally set `replenishingIndex` incorrectly with this function.

Right now, `setReplenishingIndex` does not allow the admin to set `replenishingIndex` to a value smaller than it currently is. So, if an admin were to accidentally set this value too high, then it would be impossible to set it back to a lower value (the higher the value set, the worse this issue). All of the unclaimed tokens on auctions at smaller indices would be locked forever.

## Proof of Concept
See code for `setReplenishingIndex` here: https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/AuctionParticipant.sol#L132

## Tools Used
Inspection

## Recommended Mitigation Steps
Remove the require statement on line 136, so that an admin can set the index to a smaller value.

