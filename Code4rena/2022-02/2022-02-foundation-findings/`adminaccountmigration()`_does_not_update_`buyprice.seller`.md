## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [`adminAccountMigration()` Does Not Update `buyPrice.seller`](https://github.com/code-423n4/2022-02-foundation-findings/issues/87) 

# Lines of code

https://github.com/code-423n4/2022-02-foundation/blob/main/contracts/mixins/NFTMarketReserveAuction.sol#L263-L292
https://github.com/code-423n4/2022-02-foundation/blob/main/contracts/mixins/NFTMarketBuyPrice.sol#L125-L141


# Vulnerability details

## Impact

The `adminAccountMigration()` function is called by the operator role to update all sellers' auctions. The `auction.seller` account is updated to the new address, however, the protocol fails to update `buyPrice.seller`. As a result, the protocol is put in a deadlock situation where the new address cannot cancel the auction and withdraw their NFT without the compromised account first cancelling the buy price and vice-versa. This is only recoverable if the new account is migrated back to the compromised account and then `cancelBuyPrice()` is called before migrating back.

## Proof of Concept

## Tools Used

Manual code review.

## Recommended Mitigation Steps

Consider invalidating the buy offer before account migration.

