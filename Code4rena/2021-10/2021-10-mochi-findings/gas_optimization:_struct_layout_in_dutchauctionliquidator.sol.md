## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas optimization: Struct layout in DutchAuctionLiquidator.sol](https://github.com/code-423n4/2021-10-mochi-findings/issues/54) 

# Handle

gzeon


# Vulnerability details

## Impact
Auction struct in DutchAuctionLiquidator.sol can be optimized to reduce 2 storage slot

## Proof of Concept
https://github.com/code-423n4/2021-10-mochi/blob/main/projects/mochi-core/contracts/liquidator/DutchAuctionLiquidator.sol
L18-L25: the struct can changed into
struct Auction {
        uint256 nftId;
        address vault;
        uint48 startedAt;
        uint48 boughtAt;
        uint256 collateral;
        uint256 debt;
    }
startedAt and boughtAt store block numbers, and 2^48 is be enough for a very long time.

## Tools Used
None

## Recommended Mitigation Steps
Change the struct as suggested above, also need to cast whenever startedAt and boughtAt is used.

