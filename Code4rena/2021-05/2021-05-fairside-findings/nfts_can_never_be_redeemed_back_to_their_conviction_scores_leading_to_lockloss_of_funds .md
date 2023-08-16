## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- disagree with severity
- resolved

# [NFTs can never be redeemed back to their conviction scores leading to lock/loss of funds ](https://github.com/code-423n4/2021-05-fairside-findings/issues/31) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Besides the conviction scores of users, there appears to be tracking of the FairSide protocol’s tokenized conviction score as a whole (using fscAddress = address(fairSideConviction)). This is evident in the attempted reduction of the protocol’s score when a user acquires conviction back from a NFT. However, the complementary accrual of user's conviction score to fscAddress when user tokenizes their conviction score to mint a NFT is missing in tokenizeConviction().

Because of this missing updation of conviction score to fscAddress on tokenization, there are no checkpoints written for fscAddress and there also doesn’t appear to be any initialization for bootstrapping this address’s conviction score checkpoints. As a result, the sub224() on Line350 of ERC20ConvictionScore.sol will always fail with an underflow because fscOld = 0 (because fscNum = 0) and convictionScore > 0, effectively reverting all calls to acquireConviction().

The impact is that all tokenized NFTs can never be redeemed back to their conviction scores and therefore leads to lock/loss of FSD funds for users who tokenized/sold/bought FairSide NFTs.


## Proof of Concept

1. Alice tokenizes her conviction score into a NFT. She sells that NFT to Bob who pays an amount commensurate with the conviction score captured by that NFT (as valued by the market) and any FSDs locked with the NFT. 

2. Bob then attempts to redeem the bought NFT back to the conviction score to use it on FairSide network. But the call to acquireConviction() fails. Bob is never able to redeem Alice’s NFT and has lost the funds used to buy it.

https://github.com/code-423n4/2021-05-FairSide/blob/3e9f6d40f70feb67743bdc70d7db9f5e3a1c3c96/contracts/dependencies/ERC20ConvictionScore.sol#L343-L355


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Add appropriate logic to bootstrap+initialize fscAddress’s tokenized conviction score checkpoints and update it during tokenization.

