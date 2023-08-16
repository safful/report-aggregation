## Tags

- bug
- question
- 3 (High Risk)
- sponsor confirmed
- resolved

# [Locked funds from tokenization are credited twice to user leading to protocol fund loss](https://github.com/code-423n4/2021-05-fairside-findings/issues/30) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The tokens optionally locked during tokenization are released twice on acquiring conviction back from a NFT. (The incorrect double debit of locked funds during tokenization has been filed as a separate finding because it is not necessarily related and also occurs in a different part of the code.)

When a user wants to acquire back the conviction score captured by a NFT, the FSD tokens locked, if any, are released to the user as well. However, this is incorrectly done twice. Released amount is transferred once on Line123 in _release() (via acquireConviction -> burn) of FairSideConviction.sol and again immediately after the burn on Line316 in acquireConviction() of ERC20ConvictionScore.sol.

This leads to loss of protocol funds.

## Proof of Concept

Alice tokenizes her conviction score into a NFT and locks 100 FSDs. Bob buys the NFT from Alice and acquires the conviction score back from the NFT. But instead of 100 FSDs that were supposed to be locked with the NFT, Bob receives 100+100 = 200 FSDs from FairSide protocol.

https://github.com/code-423n4/2021-05-FairSide/blob/3e9f6d40f70feb67743bdc70d7db9f5e3a1c3c96/contracts/conviction/FairSideConviction.sol#L123

https://github.com/code-423n4/2021-05-FairSide/blob/3e9f6d40f70feb67743bdc70d7db9f5e3a1c3c96/contracts/dependencies/ERC20ConvictionScore.sol#L314-L316


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Remove the redundant transfer of FSD tokens from protocol to user on Line316 in acquireConviction() of ERC20ConvictionScore.sol.

