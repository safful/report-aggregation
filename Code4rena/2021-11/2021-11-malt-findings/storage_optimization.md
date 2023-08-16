## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Storage Optimization](https://github.com/code-423n4/2021-11-malt-findings/issues/38) 

# Handle

0x1f8b


# Vulnerability details

## Impact
Cheaper storage.

## Proof of Concept

The struct AuctionData file Auction.sol is optimizable. It looks like this:

```
struct AuctionData {
  // The full amount of commitments required to return to peg
  uint256 fullRequirement;
  // total maximum desired commitments to this auction
  uint256 maxCommitments;
  // Quantity of sale currency committed to this auction
  uint256 commitments;
  // Malt purchased and burned using current commitments
  uint256 maltPurchased;
  // Desired starting price for the auction
  uint256 startingPrice;
  // Desired lowest price for the arbitrage token
  uint256 endingPrice;
  // Price of arbitrage tokens at conclusion of auction. This is either
  // when the duration elapses or the maxCommitments is reached
  uint256 finalPrice;
  // The peg price for the liquidity pool
  uint256 pegPrice;
  // Time when auction started
  uint256 startingTime;
  uint256 endingTime;
  // Is the auction currently accepting commitments?
  bool active;
  // The reserve ratio at the start of the auction
  uint256 preAuctionReserveRatio;
  // Has this auction been finalized? Meaning any additional stabilizing
  // has been done
  bool finalized;
  // The amount of arb tokens that have been executed and are now claimable
  uint256 claimableTokens;
  // The finally calculated realBurnBudget
  uint256 finalBurnBudget;
  // The amount of Malt purchased with realBurnBudget
  uint256 finalPurchased;
  // A map of all commitments to this auction by specific accounts
  mapping(address => AccountCommitment) accountCommitments;
}
```
But `active` and `finalized`, the unique boolean values, should be together, otherwise they will spend two slots instead of one.
```
  uint256 preAuctionReserveRatio;
  bool active;
  bool finalized;
```

## Tools Used

Manual review

## Recommended Mitigation Steps

