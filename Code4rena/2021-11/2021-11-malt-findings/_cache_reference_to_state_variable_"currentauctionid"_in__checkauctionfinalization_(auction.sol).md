## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [ Cache Reference To State Variable "currentAuctionID" in _checkAuctionFinalization (Auction.sol)](https://github.com/code-423n4/2021-11-malt-findings/issues/89) 

# Handle

ye0lde


# Vulnerability details

## Impact

Caching the references to "currentAuctionID" will decrease gas usage. 

## Proof of Concept

The state variable "currentAuctionID" is read 7 times in function "_checkAuctionFinalization" here:
https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/Auction.sol#L746-L762

## Tools Used
Visual Studio Code, Remix

## Recommended Mitigation Steps

I suggest making the changes below to cache the variable:
  
```
  function _checkAuctionFinalization(bool isInternal) internal {

    uint256 currentId = currentAuctionId;
    if (isInternal && !isAuctionFinished(currentId)) {
      // Auction is still in progress after internal auction purchasing.
      _resetAuctionMaxCommitments();
    }

    if (isAuctionFinished(currentId)) {
      if (auctionActive(currentId)) {
        _endAuction(currentId);
      }

      if (!isAuctionFinalized(currentId)) {
        _finalizeAuction(currentId);
      }
      currentAuctionId = currentId + 1;
    }
  }
```


