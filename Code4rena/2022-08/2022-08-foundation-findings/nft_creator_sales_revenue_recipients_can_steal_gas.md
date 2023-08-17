## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [NFT creator sales revenue recipients can steal gas](https://github.com/code-423n4/2022-08-foundation-findings/issues/165) 

# Lines of code

https://github.com/code-423n4/2022-08-foundation/blob/792e00df429b0df9ee5d909a0a5a6e72bd07cf79/contracts/mixins/shared/MarketFees.sol#L130


# Vulnerability details

## Impact

Selling a NFT with `NFTDropMarketFixedPriceSale.mintFromFixedPriceSale` distributes the revenue from the sale to various recipients with the `MarketFees._distributeFunds` function.

Recipients:

- NFT creator(s)
- NFT seller
- Protocol
- Buy referrer (optional)

It is possible to have multiple NFT creators. Sale revenue will be distributed to each NFT creator address. Revenue distribution is done by calling `SendValueWithFallbackWithdraw._sendValueWithFallbackWithdraw` and providing an appropriate gas limit to prevent consuming too much gas. For the revenue distribution to the seller, protocol and the buy referrer, a gas limit of `SEND_VALUE_GAS_LIMIT_SINGLE_RECIPIENT = 20_000` is used. However, for the creators, a limit of `SEND_VALUE_GAS_LIMIT_MULTIPLE_RECIPIENTS = 210_000` is used. This higher amount of gas is used if `PercentSplitETH` is used as a recipient.

A maximum of `MAX_ROYALTY_RECIPIENTS = 5` NFT creator recipients are allowed.

For example, a once honest NFT collection and its 5 royalty creator recipients could turn "malicious" and could "steal" gas from NFT buyers on each NFT sale and therefore grief NFT sales. On each NFT sell, the 5 creator recipients (smart contracts) could consume the full amount of `SEND_VALUE_GAS_LIMIT_MULTIPLE_RECIPIENTS = 210_000` forwarded gas. Totalling `5 * 210_000 = 1_050_000` gas. With a gas price of e.g. `20 gwei`, this equals to additional gas costs of `21_000_000 gwei = 0.028156 eth`, with a `ETH` price of `2000`, this would total to ~`56.31 $` additional costs.

## Proof of Concept

[mixins/shared/MarketFees.sol#L130](https://github.com/code-423n4/2022-08-foundation/blob/792e00df429b0df9ee5d909a0a5a6e72bd07cf79/contracts/mixins/shared/MarketFees.sol#L130)

```solidity
/**
  * @notice Distributes funds to foundation, creator recipients, and NFT owner after a sale.
  */
function _distributeFunds(
  address nftContract,
  uint256 tokenId,
  address payable seller,
  uint256 price,
  address payable buyReferrer
)
  internal
  returns (
    uint256 totalFees,
    uint256 creatorRev,
    uint256 sellerRev
  )
{
  address payable[] memory creatorRecipients;
  uint256[] memory creatorShares;

  uint256 buyReferrerFee;
  (totalFees, creatorRecipients, creatorShares, sellerRev, buyReferrerFee) = _getFees(
    nftContract,
    tokenId,
    seller,
    price,
    buyReferrer
  );

  // Pay the creator(s)
  unchecked {
    for (uint256 i = 0; i < creatorRecipients.length; ++i) {
      _sendValueWithFallbackWithdraw(
        creatorRecipients[i],
        creatorShares[i],
        SEND_VALUE_GAS_LIMIT_MULTIPLE_RECIPIENTS // @audit-info A higher amount of gas is forwarded to creator recipients
      );
      // Sum the total creator rev from shares
      // creatorShares is in ETH so creatorRev will not overflow here.
      creatorRev += creatorShares[i];
    }
  }

  // Pay the seller
  _sendValueWithFallbackWithdraw(seller, sellerRev, SEND_VALUE_GAS_LIMIT_SINGLE_RECIPIENT);

  // Pay the protocol fee
  _sendValueWithFallbackWithdraw(getFoundationTreasury(), totalFees, SEND_VALUE_GAS_LIMIT_SINGLE_RECIPIENT);

  // Pay the buy referrer fee
  if (buyReferrerFee != 0) {
    _sendValueWithFallbackWithdraw(buyReferrer, buyReferrerFee, SEND_VALUE_GAS_LIMIT_SINGLE_RECIPIENT);
    emit BuyReferralPaid(nftContract, tokenId, buyReferrer, buyReferrerFee, 0);
    unchecked {
      // Add the referrer fee back into the total fees so that all 3 return fields sum to the total price for events
      totalFees += buyReferrerFee;
    }
  }
}
```

## Tools Used

Manual review

## Recommended mitigation steps

Consider only providing a higher amount of gas (`SEND_VALUE_GAS_LIMIT_MULTIPLE_RECIPIENTS`) for the first creator recipient. For all following creator recipients, only forward the reduced amount of gas `SEND_VALUE_GAS_LIMIT_SINGLE_RECIPIENT`.
