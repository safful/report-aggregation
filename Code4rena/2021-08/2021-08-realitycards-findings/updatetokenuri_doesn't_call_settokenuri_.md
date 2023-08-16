## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- Resolved

# [updateTokenURI doesn't call setTokenURI ](https://github.com/code-423n4/2021-08-realitycards-findings/issues/12) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function updateTokenURI of RCFactory.sol doesn't update the uris of RCNftHubL2.
E.g. it doesn't call setTokenURI to try and update the already created NFT's.
This way the URIs of already minted tokens are not updated.

## Proof of Concept
// https://github.com/code-423n4/2021-08-realitycards/blob/main/contracts/RCFactory.sol#L453
 function updateTokenURI(

// https://github.com/code-423n4/2021-08-realitycards/blob/main/contracts/nfthubs/RCNftHubL2.sol#L101
function setTokenURI(uint256 _tokenId, string calldata _tokenURI) external onlyUberOwner {
        _setTokenURI(_tokenId, _tokenURI);
    }

## Tools Used

## Recommended Mitigation Steps
Also call setTokenURI of RCNftHubL2
Or restrict updateTokenURI to the phase where no NFT's are minted yet.
Or at least add comments to updateTokenURI 

