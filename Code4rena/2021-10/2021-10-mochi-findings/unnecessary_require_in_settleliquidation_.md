## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Unnecessary require in settleLiquidation ](https://github.com/code-423n4/2021-10-mochi-findings/issues/71) 

# Handle

harleythedog


# Vulnerability details

## Impact
On line 100 of DutchAuctionLiquidator.sol (within settleLiquidation), there is a require statement for auction.boughtAt == 0. This is already checked on line 121 within the "buy" function, and this is the only function that can possibly call settleLiquidation, so this require statement always passes. Removing it would save gas.

## Proof of Concept
Link to require statement here: https://github.com/code-423n4/2021-10-mochi/blob/main/projects/mochi-core/contracts/liquidator/DutchAuctionLiquidator.sol#:~:text=require(auction.boughtAt%20%3D%3D%200%2C%20%22liquidated%22)%3B

## Tools Used
Manual inspection.

## Recommended Mitigation Steps
Remove unnecessary require statement described above to save gas.

