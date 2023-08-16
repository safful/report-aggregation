## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Auction:claimArbitrage no underflow checks needed](https://github.com/code-423n4/2021-11-malt-findings/issues/149) 

# Handle

GiveMeTestEther


# Vulnerability details

# Vulnerability details

## Impact
The else block on L241 satisfies amountTokens <= unclaimedArbTokens and therefore we don't need to do a safe subtraction (underflow check).

The else block on L247 satisfies amountTokens <= claimableArbitrageRewards and therefore we don't need to do a safe subtraction (underflow check).




## Proof of Concept
https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/Auction.sol#L216
https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/Auction.sol#L247

## Tools Used

## Recommended Mitigation Steps
- rewrite L241 as: unclaimedArbTokens = unclaimedArbTokens - amountTokens;
- rewrite L274 as: claimableArbitrageRewards = claimableArbitrageRewards - amountTokens-;


