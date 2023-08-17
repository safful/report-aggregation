## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- resolved
- sponsor confirmed
- selected for report
- responded

# [Source contract can steal NFTs from users](https://github.com/code-423n4/2022-10-holograph-findings/issues/290) 

# Lines of code

https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/contracts/enforcer/HolographERC721.sol#L500
https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/contracts/enforcer/HolographERC721.sol#L577


# Vulnerability details

## Impact
A source contract can burn and transfer NFTs of users without their permission.
## Proof of Concept
Every Holographed ERC721 collection is paired with a source contract, which is the user created contract that's extended by the Holographed ERC721 contract ([HolographFactory.sol#L234-L246](https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/contracts/HolographFactory.sol#L234-L246)). A source contract, however, has excessive privileges in the Holographed ERC721. Specifically, it can burn and transfer users' NFTs without their approval ([HolographERC721.sol#L500](https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/contracts/enforcer/HolographERC721.sol#L500), [HolographERC721.sol#L577](https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/contracts/enforcer/HolographERC721.sol#L577)):
```solidity
function sourceBurn(uint256 tokenId) external onlySource {
  address wallet = _tokenOwner[tokenId];
  _burn(wallet, tokenId);
}

function sourceTransfer(address to, uint256 tokenId) external onlySource {
  address wallet = _tokenOwner[tokenId];
  _transferFrom(wallet, to, tokenId);
}
```

While this might be desirable for extensibility and flexibility, this puts users at the risk of being robbed by the source contract owner or a hacker who hacked the source contract owner's key.
## Tools Used
Manual review
## Recommended Mitigation Steps
Consider removing the `sourceBurn` and `sourceTransfer` functions of `HolographERC721` and requiring user approval to transfer or burn their tokens (`burn` and `safeTransferFrom` can be called by a source contract instead of `sourceBurn` and `sourceTransfer`).