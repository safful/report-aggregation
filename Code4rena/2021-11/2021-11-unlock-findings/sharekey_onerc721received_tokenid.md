## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [shareKey onERC721Received tokenId](https://github.com/code-423n4/2021-11-unlock-findings/issues/91) 

# Handle

HardlyDifficult


# Vulnerability details

## Impact
A contract implementing `ERC721TokenReceiver` is called with a tokenId that was not sent to that address when `shareKey` is used. If the `onERC721Received` implementation included any logic which assumed ownership it may fail, e.g. checking `ownerOf`, `balanceOf` or performing a task such as `transferFrom` to forward the asset to another destination.

## Proof of Concept
`shareKey` accepts a `_tokenId` as the source of expiration time to share. It then either mints a new token for the target account or adds time to their existing key. Either way the receiver has a different tokenId than the one that was passed to the `shareKey` function.

## Tools Used
n/a

## Recommended Mitigation Steps
Change https://github.com/code-423n4/2021-11-unlock/blob/52f3f3d0524dda28aea327181c3479d85782007b/smart-contracts/contracts/mixins/MixinTransfer.sol#L106

from:
`require(_checkOnERC721Received(keyOwner, _to, _tokenId, ''), 'NON_COMPLIANT_ERC721_RECEIVER');`

to:
`require(_checkOnERC721Received(keyOwner, _to, idTo, ''), 'NON_COMPLIANT_ERC721_RECEIVER');`

