## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- Resolved

# [Inaccurate Comment](https://github.com/code-423n4/2021-08-realitycards-findings/issues/16) 

# Handle

leastwood


# Vulnerability details

## Impact
This issue has no direct security implications, however, there may be some confusion when understanding what the `RCFactory.createMarket()` function actually does.

## Proof of Concept
https://github.com/code-423n4/2021-08-realitycards/blob/main/contracts/RCFactory.sol#L625

## Tools Used

Manual code review

## Recommended Mitigation Steps

Update the line (linked above) to include the `SAFE_MODE` option outline in the `enum` type in `IRCMarket.sol`. For example, the line `/// @param _mode 0 = normal, 1 = winner takes all` could be updated to `/// @param _mode 0 = normal, 1 = winner takes all, 2 = SAFE_MODE`

