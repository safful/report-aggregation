## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Anyone can set the `baseRatePerYear` after the `updateFrequency` has passed](https://github.com/code-423n4/2022-06-canto-findings/issues/22) 

# Lines of code

https://github.com/Plex-Engineer/lending-market/blob/755424c1f9ab3f9f0408443e6606f94e4f08a990/contracts/NoteInterest.sol#L118-L129


# Vulnerability details

## Impact
The `updateBaseRate()` function is public and lacks access control, so anyone can set the critical variable `baseRatePerYear` once the block delta has surpassed the `updateFrequency` variable. This will have negative effects on the borrow and supply rates used anywhere else in the protocol.

The updateFrequency is explained to default to 24 hours per the comments, so this vulnerability will be available every day. Important to note, the admin can fix the `baseRatePerYear` by calling the admin-only `_setBaseRatePerYear()` function. However, calling this function does not set the `lastUpdateBlock` so users will still be able to change the rate back after the 24 hours waiting period from the previous change. 

## Proof of Concept
```
    function updateBaseRate(uint newBaseRatePerYear) public {
        // check the current block number
        uint blockNumber = block.number;
        uint deltaBlocks = blockNumber.sub(lastUpdateBlock);


        if (deltaBlocks > updateFrequency) {
            // pass in a base rate per year
            baseRatePerYear = newBaseRatePerYear;
            lastUpdateBlock = blockNumber;
            emit NewInterestParams(baseRatePerYear);
        }
    }
```

## Tools Used
Manual review.

## Recommended Mitigation Steps
I have trouble understanding the intention of this function. It appears that the rate should only be able to be set by the admin, so the `_setBaseRatePerYear()` function seems sufficient. Otherwise, add access control for only trusted parties.

