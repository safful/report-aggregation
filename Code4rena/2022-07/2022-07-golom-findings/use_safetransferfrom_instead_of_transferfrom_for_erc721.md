## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- resolved
- sponsor confirmed
- selected-for-report

# [Use safeTransferFrom Instead of transferFrom for ERC721](https://github.com/code-423n4/2022-07-golom-findings/issues/342) 

# Lines of code

https://github.com/code-423n4/2022-07-golom/blob/7bbb55fca61e6bae29e57133c1e45806cbb17aa4/contracts/core/GolomTrader.sol#L236


# Vulnerability details

## Impact
Use of transferFrom method for ERC721 transfer is discouraged and recommended to use safeTransferFrom 
whenever possible by OpenZeppelin.
This is because transferFrom() cannot check whether the receiving address know how to handle ERC721 tokens.

In the function shown at below PoC, ERC721 token is sent to msg.sender with the transferFrom method.
If this msg.sender is a contract and is not aware of incoming ERC721 tokens, the sent token could be locked up in
the contract forever.

Reference: https://docs.openzeppelin.com/contracts/3.x/api/token/erc721

## Proof of Concept
```
GolomTrader.sol:236:            ERC721(o.collection).transferFrom(o.signer, receiver, o.tokenId);
```

## Tools Used
Manual Analysis

## Recommended Mitigation Steps
I recommend to call the safeTransferFrom() method instead of transferFrom() for NFT transfers.