## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Use bps uniformly](https://github.com/code-423n4/2021-11-malt-findings/issues/30) 

# Handle

tabish


# Vulnerability details

## Impact
Detailed description of the impact of this finding.
Some variables are in form of bps https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/Auction.sol#L90 
while some are not https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/Auction.sol#L91 https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/Auction.sol#L92 
As a good programming practice, should use bps everywhere if you accepting it as a unit 


