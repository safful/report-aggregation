## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Auction parameters can be changed during ongoing auction](https://github.com/code-423n4/2022-09-nouns-builder-findings/issues/450) 

# Lines of code

https://github.com/code-423n4/2022-09-nouns-builder/blob/7e9fddbbacdd7d7812e912a369cfd862ee67dc03/src/auction/Auction.sol#L307-L335


# Vulnerability details

## Impact

The auction parameters can be changed anytime, even during ongoing auctions, and take effect immediately. Users may need time to react to the changes. The impacts maybe followings:
- some sudden changes may cause bidder's transaction fail, such as `setReservePrice()` and `setMinimumBidIncrement()`
- some changes may change users expectation about the auction, such as `setDuration()` and `setTimeBuffer()`, with different time parameters, bidders will use different strategy


## Proof of Concept

src/auction/Auction.sol
```solidity
    function setDuration(uint256 _duration) external onlyOwner {
        settings.duration = SafeCast.toUint40(_duration);

        emit DurationUpdated(_duration);
    }

    function setReservePrice(uint256 _reservePrice) external onlyOwner {
        settings.reservePrice = _reservePrice;

        emit ReservePriceUpdated(_reservePrice);
    }

    function setTimeBuffer(uint256 _timeBuffer) external onlyOwner {
        settings.timeBuffer = SafeCast.toUint40(_timeBuffer);

        emit TimeBufferUpdated(_timeBuffer);
    }

    function setMinimumBidIncrement(uint256 _percentage) external onlyOwner {
        settings.minBidIncrement = SafeCast.toUint8(_percentage);

        emit MinBidIncrementPercentageUpdated(_percentage);
    }```


## Tools Used
Manual analysis.

## Recommended Mitigation Steps

- do not apply changed parameters on ongoing auctions 
- add a timelock for the changes

