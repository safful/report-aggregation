## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- resolved

# [Making isMarketApproved False on an operational market will lock NFTs to L2](https://github.com/code-423n4/2021-06-realitycards-findings/issues/76) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Once market is approved and operational, changing approval to false should not be allowed or else it will prevent NFTs from being withdrawn to mainnet. All other Governor controlled variables are used during market creation and not thereafter, except this one. The other onlyGovernors functions only affect state before market creation but this one affects after creation.

## Proof of Concept

https://github.com/code-423n4/2021-06-realitycards/blob/86a816abb058cc0ed9b6f5c4a8ad146f22b8034c/contracts/RCFactory.sol#L382-L391

https://github.com/code-423n4/2021-06-realitycards/blob/86a816abb058cc0ed9b6f5c4a8ad146f22b8034c/contracts/RCMarket.sol#L326-L330


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Once market is approved and operational, changing approval to false should not be allowed.

