## Tags

- bug
- QA (Quality Assurance)
- disagree with severity
- sponsor confirmed

# [QA report](https://github.com/code-423n4/2022-02-foundation-findings/issues/9) 

# Lines of code

https://github.com/code-423n4/2022-02-foundation/blob/main/contracts/mixins/NFTMarketOffer.sol#L150


# Vulnerability details

## Impact
In NFTMarketOffer.sol the adminCancelOffers() function has comments above it that mention 

"tokenIds The ids of the NFTs to cancel. This must be the same length as `nftContracts`"

This means that both the tokensIds and nftContracts arrays must be the same length but this is not required in the code of the function itself which can lead to the function failing. 

## Proof of Concept
https://github.com/code-423n4/2022-02-foundation/blob/main/contracts/mixins/NFTMarketOffer.sol#L150

## Tools Used
Manual code review 

## Recommended Mitigation Steps
Add to adminCancelOffers() function:  
require(nftContracts.length == tokenIds.length, Arrays must be same length");

