## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Auction:amendAccountParticipation no underflow check needed](https://github.com/code-423n4/2021-11-malt-findings/issues/151) 

# Handle

GiveMeTestEther


# Vulnerability details

## Impact
Else block on L805 satisfies amountArbTokens <= unclaimedArbTokens and therefore no safe subtraction (underflow check) is needed (saves gas).

## Proof of Concept
https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/Auction.sol#L786

## Tools Used

## Recommended Mitigation Steps
- rewrite L805 as: unclaimedArbTokens = unclaimedArbTokens - amountArbTokens;

