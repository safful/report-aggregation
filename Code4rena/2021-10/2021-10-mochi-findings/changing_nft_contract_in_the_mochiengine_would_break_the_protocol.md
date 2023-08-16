## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Changing NFT contract in the MochiEngine would break the protocol](https://github.com/code-423n4/2021-10-mochi-findings/issues/63) 

# Handle

jonah1005


# Vulnerability details

## Impact
MochiEngine allows the operator to change the NFT contract.
[MochiEngine.sol#L91-L93](https://github.com/code-423n4/2021-10-mochi/blob/main/projects/mochi-core/contracts/MochiEngine.sol#L91-L93)
All the vaults would point to a different NFT address. As a result, users would not be access their positions. The entire protocol would be broken.

IMHO, A function that would break the entire protocol shouldn't exist.

I consider this is a high-risk issue.
## Proof of Concept
[MochiEngine.sol#L91-L93](https://github.com/code-423n4/2021-10-mochi/blob/main/projects/mochi-core/contracts/MochiEngine.sol#L91-L93)

## Tools Used
None
## Recommended Mitigation Steps
Remove the function.


