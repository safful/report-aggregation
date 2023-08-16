## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- resolved

# [totalNftMintCount can be replaced with ERC721 totalSupply()](https://github.com/code-423n4/2021-06-realitycards-findings/issues/134) 

# Handle

pauliax


# Vulnerability details

## Impact
I can't find a reason why totalNftMintCount in Factory can't be replaced with ERC721 totalSupply() to make it less error-prone. As nfthub.mint issues a new token it should automatically increment totalSupply and this assignment won't be needed:
      totalNftMintCount = totalNftMintCount + _tokenURIs.length;
Also in function setNftHubAddress you need to manually set _newNftMintCount if you want to change nfthub so an invalid value may crash the system. totalSupply() will eliminate totalNftMintCount and make the system more robust.

## Recommended Mitigation Steps
Replace totalNftMintCount with nfthub totalSupply() in Factory contract.

