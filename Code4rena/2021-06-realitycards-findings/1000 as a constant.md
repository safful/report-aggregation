## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- resolved

# [1000 as a constant](https://github.com/code-423n4/2021-06-realitycards-findings/issues/10) 

# Handle

gpersoon


# Vulnerability details

## Impact
A value of 1000 is used to indicate 100%. This value is hardcoded on several places.
It's saver to use a constant, to prevent mistakes in future updates.

## Proof of Concept
.\RCFactory.sol:    /// @dev in basis points (so 1000 = 100%)
.\RCFactory.sol:                1000,
.\RCMarket.sol:                (((uint256(1000) - artistCut) - creatorCut) - affiliateCut) -
.\RCMarket.sol:            ((((uint256(1000) - artistCut) - affiliateCut) - cardAffiliateCut) -
.\RCMarket.sol:            _winningsToTransfer = (totalRentCollected * winnerCut) / (1000);
.\RCMarket.sol:                (1000);
.\RCMarket.sol:            _remainingPot = (totalRentCollected * _remainingCut) / (1000);
.\RCMarket.sol:            ((uint256(1000) - artistCut) - affiliateCut) - cardAffiliateCut;
.\RCMarket.sol:            (_rentCollected * _remainingCut) / (1000);
.\RCMarket.sol:            (rentCollectedPerCard[_card] * cardAffiliateCut) / (1000);
.\RCMarket.sol:            uint256 _payment = (totalRentCollected * _cut) / (1000);

## Tools Used
grep

## Recommended Mitigation Steps
Replace 1000 with a constant.


