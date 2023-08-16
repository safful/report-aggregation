## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [AuctionParticipant.sol: `replenishingIndex` incrementing should be improved](https://github.com/code-423n4/2021-11-malt-findings/issues/85) 

# Handle

harleythedog


# Vulnerability details

## Impact
In AuctionParticipant.sol, the `claim` function is called to claim arb tokens from auctions the participant has entered. This is achieved through the global variable `replenishingIndex` which keeps track of which auction `claim` should be claiming from next. The logic for incrementing `replenishingIndex` is at the end of claim. 

I agree with the current logic at the end of the function. The comment on lines 96/97 says "Don't increment replenishingIndex if replenishingAuctionId == auctionId as claimable could be 0 due to the debt not being 100% replenished". Notice the keyword "could" - it is possible that replenishingAuctionId == auctionId but we will never be able to claim any more arb tokens from this contract, and in this case `replenishingIndex` will NOT be incremented.

In this case, all subsequent calls to `claim` will simply do nothing. Line 77 will have `claimableTokens` be 0, and then the function will immediately return since it thinks it needs to wait longer to get more tokens, which will never happen. In this case, a manual intervention by an admin would be required to set `replenishingIndex', which is obviously annoying and should be avoided. Since `claim` is an external function, a malicious user/troll could intentionally call `claim` at the worst times to trigger this issue to happen. In this case, manual intervention would be required quite often.

The following logic should be added immediately after line 77 to account for this issue:

if (claimableTokens == 0 && replenishingId > auctionId) { // in this case, we will never receive any more tokens from this auction
    replenishingIndex = replenishingIndex + 1;
    auctionId = auctionIds[replenishingIndex];
}

// retry check for 0 claimable amount

## Proof of Concept
See the code for `claim` here: https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/AuctionParticipant.sol#L65

Other than manual intervention, the only place where `replenishingIndex` is set is at the end of `claim`.

## Tools Used
Inspection

## Recommended Mitigation Steps
Add the code described above.

