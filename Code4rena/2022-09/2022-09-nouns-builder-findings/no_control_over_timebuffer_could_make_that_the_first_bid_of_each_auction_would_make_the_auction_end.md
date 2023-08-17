## Tags

- bug
- QA (Quality Assurance)
- sponsor confirmed

# [No control over timeBuffer could make that the first bid of each auction would make the auction end](https://github.com/code-423n4/2022-09-nouns-builder-findings/issues/359) 

# Lines of code

https://github.com/code-423n4/2022-09-nouns-builder/blob/main/src/auction/Auction.sol#L147-L150
https://github.com/code-423n4/2022-09-nouns-builder/blob/main/src/auction/Auction.sol#L146
https://github.com/code-423n4/2022-09-nouns-builder/blob/main/src/auction/Auction.sol#L323-L327


# Vulnerability details

## Impact
There is an [unchecked block](https://github.com/code-423n4/2022-09-nouns-builder/blob/main/src/auction/Auction.sol#L147-L150) that in case timeBuffer is sufficiently large, would make the sum overflow and set the endTime of the auction in the past, making the auction end automatically. The developers are aware of this, that's why they have [this comment](https://github.com/code-423n4/2022-09-nouns-builder/blob/main/src/auction/Auction.sol#L146). But I think it's not a matter of realistically overflowing, it could be set to a value large enough by mistake. It's not worth it to not validate the value of the time buffer, because the consequences could be devastating.

The best option would be to validate in function [setTimeBuffer](https://github.com/code-423n4/2022-09-nouns-builder/blob/main/src/auction/Auction.sol#L323-L327) that the timeBuffer cannot be set to a large value.

## Tools Used
Manual analysis

## Recommended Mitigation Steps
Change function setTimeBuffer with this:

```
    function setTimeBuffer(uint256 _timeBuffer) external onlyOwner {
        require(_timeBuffer < 31536000, "TimeBuffer: too big");

        settings.timeBuffer = SafeCast.toUint40(_timeBuffer);

        emit TimeBufferUpdated(_timeBuffer);
    }
```
I supposed that **timeBuffer** should be less than one year (probably much less), so I compared here with the number of seconds in a year.