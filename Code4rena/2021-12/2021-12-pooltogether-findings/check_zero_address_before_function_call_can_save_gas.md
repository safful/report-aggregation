## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Check Zero Address Before Function Call Can Save Gas](https://github.com/code-423n4/2021-12-pooltogether-findings/issues/35) 

# Handle

Meta0xNull


# Vulnerability details

## Impact
<code>require(_to != address(0), "TwabRewards/recipient-not-zero-address");</code>

Check Zero Address Before Function Call eg. _requirePromotionActive() Can Save Gas.

## Proof of Concept
https://github.com/pooltogether/v4-periphery/blob/b520faea26bcf60371012f6cb246aa149abd3c7d/contracts/TwabRewards.sol#L128

## Tools Used
Manual Review

## Recommended Mitigation Steps
Move Zero Address Check to Line L125:
https://github.com/pooltogether/v4-periphery/blob/b520faea26bcf60371012f6cb246aa149abd3c7d/contracts/TwabRewards.sol#L125

