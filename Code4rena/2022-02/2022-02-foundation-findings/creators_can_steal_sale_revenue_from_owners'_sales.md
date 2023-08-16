## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Creators can steal sale revenue from owners' sales](https://github.com/code-423n4/2022-02-foundation-findings/issues/30) 

# Lines of code

https://github.com/code-423n4/2022-02-foundation/blob/a03a7e198c1dfffb1021c0e8ec91ba4194b8aa12/contracts/mixins/NFTMarketCreators.sol#L158-L160
https://github.com/code-423n4/2022-02-foundation/blob/a03a7e198c1dfffb1021c0e8ec91ba4194b8aa12/contracts/mixins/NFTMarketCreators.sol#L196-L198
https://github.com/code-423n4/2022-02-foundation/blob/a03a7e198c1dfffb1021c0e8ec91ba4194b8aa12/contracts/mixins/NFTMarketCreators.sol#L97-L99


# Vulnerability details

According to the `README.md`
> All sales in the Foundation market will pay the creator 10% royalties on secondary sales. This is not specific to NFTs minted on Foundation, it should work for any NFT. If royalty information was not defined when the NFT was originally deployed, it may be added using the Royalty Registry which will be respected by our market contract.

https://github.com/code-423n4/2022-02-foundation/blob/4d8c8931baffae31c7506872bf1100e1598f2754/README.md?plain=1#L21

Using the Royalty Registry an owner can decide to change the royalty information right before the sale is complete, affecting who gets what.

## Impact
By updating the registry to include the seller as one of the royalty recipients, the creator can steal the sale price minus fees. This is because if code finds that the seller is a royalty recipient the royalties are all passed to the creator regardless of whether the owner is the seller or not.

## Proof of Concept
```solidity
          // 4th priority: getRoyalties override
          if (recipients.length == 0 && nftContract.supportsERC165Interface(type(IGetRoyalties).interfaceId)) {
            try IGetRoyalties(nftContract).getRoyalties{ gas: READ_ONLY_GAS_LIMIT }(tokenId) returns (
              address payable[] memory _recipients,
              uint256[] memory recipientBasisPoints
            ) {
              if (_recipients.length > 0 && _recipients.length == recipientBasisPoints.length) {
                bool hasRecipient;
                for (uint256 i = 0; i < _recipients.length; ++i) {
                  if (_recipients[i] != address(0)) {
                    hasRecipient = true;
                    if (_recipients[i] == seller) {
                      return (_recipients, recipientBasisPoints, true);
```
https://github.com/code-423n4/2022-02-concur/blob/72b5216bfeaa7c52983060ebfc56e72e0aa8e3b0/contracts/MasterChef.sol#L127-L154

When `true` is returned as the final return value above, the following code leaves `ownerRev` as zero because `isCreator` is `true`
```solidity
      uint256 ownerRev
    )
  {
    bool isCreator;
    (creatorRecipients, creatorShares, isCreator) = _getCreatorPaymentInfo(nftContract, tokenId, seller);

    // Calculate the Foundation fee
    uint256 fee;
    if (isCreator && !_nftContractToTokenIdToFirstSaleCompleted[nftContract][tokenId]) {
      fee = PRIMARY_FOUNDATION_FEE_BASIS_POINTS;
    } else {
      fee = SECONDARY_FOUNDATION_FEE_BASIS_POINTS;
    }

    foundationFee = (price * fee) / BASIS_POINTS;

    if (creatorRecipients.length > 0) {
      if (isCreator) {
        // When sold by the creator, all revenue is split if applicable.
        creatorRev = price - foundationFee;
      } else {
        // Rounding favors the owner first, then creator, and foundation last.
        creatorRev = (price * CREATOR_ROYALTY_BASIS_POINTS) / BASIS_POINTS;
        ownerRevTo = seller;
        ownerRev = price - foundationFee - creatorRev;
      }
    } else {
      // No royalty recipients found.
      ownerRevTo = seller;
      ownerRev = price - foundationFee;
    }
  }
```

In addition, if the index of the seller in `_recipients` is greater than `MAX_ROYALTY_RECIPIENTS_INDEX`, then the seller is omitted from the calculation and gets zero (`_sendValueWithFallbackWithdraw()` doesn't complain when it sends zero)
```solidity
        uint256 maxCreatorIndex = creatorRecipients.length - 1;
        if (maxCreatorIndex > MAX_ROYALTY_RECIPIENTS_INDEX) {
          maxCreatorIndex = MAX_ROYALTY_RECIPIENTS_INDEX;
        }
```
https://github.com/code-423n4/2022-02-foundation/blob/4d8c8931baffae31c7506872bf1100e1598f2754/contracts/mixins/NFTMarketFees.sol#L76-L79

This issue does a lot of damage because the creator can choose whether and when to apply it on a sale-by-sale basis. Two other similar, but separate, exploits are available for the other blocks in `_getCreatorPaymentInfo()` that return arrays but they either require a malicious NFT implementation or can only specify a static seller for which this will affect things. In all cases, not only may the seller get zero dollars for the sale, but they'll potentially owe a lot of taxes based on the 'sale' price. The attacker may or may not be the creator - creators can be bribed with kickbacks.

## Tools Used
Code inspection

## Recommended Mitigation Steps
Always calculate owner/seller revenue separately from royalty revenue


