## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [`_calculateMaltRequiredForExit` Uses Spot Price To Calculate Malt Quantity In `exitEarly`](https://github.com/code-423n4/2021-11-malt-findings/issues/215) 

# Handle

leastwood


# Vulnerability details

## Impact

`_calculateMaltRequiredForExit` in `AuctionEscapeHatch` currently uses Malt's spot price to calculate the quantity to return to the exiting user. This spot price simply tracks the Uniswap pool's reserves which can easily be manipulated via a flash loan attack to extract funds from the protocol.

## Proof of Concept

https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/AuctionEscapeHatch.sol#L65-L92
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/AuctionEscapeHatch.sol#L193
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/DexHandlers/UniswapHandler.sol#L80-L109
https://shouldiusespotpriceasmyoracle.com/

## Tools Used

Manual code review.

## Recommended Mitigation Steps

Consider implementing/integrating a TWAP oracle to track the price of Malt.

