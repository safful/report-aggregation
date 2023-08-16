## Tags

- bug
- 0 (Non-critical)
- resolved
- sponsor confirmed

# [Event for merge](https://github.com/code-423n4/2022-01-xdefi-findings/issues/197) 

# Handle

0xsanson


# Vulnerability details

Not an issue.

I noticed that the `merge` function doesn't have an event associated with it. Depending on the kind of offchain analysis/tools you will end up using, an event here may turn up useful to know which NFTs got merged together into a new one.

## Recommended Mitigation Steps
Add an event which contains `uint256[] memory tokenIds_` and `tokenId_`.

