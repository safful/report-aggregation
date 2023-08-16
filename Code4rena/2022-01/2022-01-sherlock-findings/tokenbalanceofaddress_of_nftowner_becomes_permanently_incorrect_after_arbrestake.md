## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [tokenBalanceOfAddress of nftOwner becomes permanently incorrect after arbRestake](https://github.com/code-423n4/2022-01-sherlock-findings/issues/109) 

# Handle

hyh


# Vulnerability details

## Impact

Sucessfull `arbRestake` performs `_redeemShares` for `arbRewardShares` amount to extract the arbitrager reward. This effectively reduces shares accounted for an NFT, but leaves untouched the `addressShares` of an `nftOwner`.

As a result the `tokenBalanceOfAddress` function will report an old balance that existed before arbitrager reward was slashed away. This will persist if the owner will transfer the NFT to someone else as its new reduced shares value will be subtracted from `addressShares` in `_beforeTokenTransfer`, leaving the arbitrage removed shares permanently in `addressShares` of the NFT owner, essentially making all further reporting of his balance incorrectly inflated by the cumulative arbitrage reward shares from all arbRestakes happened to the owner's NFTs.

## Proof of Concept

`arbRestake` redeems `arbRewardShares`, which are a part of total shares of an NFT:

https://github.com/code-423n4/2022-01-sherlock/blob/main/contracts/Sherlock.sol#L673


This will effectively reduce the `stakeShares`:

https://github.com/code-423n4/2022-01-sherlock/blob/main/contracts/Sherlock.sol#L491

But there is no mechanics in place to reduce `addressShares` of the owner apart from mint/burn/transfer, so `addressShares` will still correspond to NFT shares before arbitrage. This discrepancy will be accumulated further with arbitrage restakes.


## Recommended Mitigation Steps

Add a flag to `_redeemShares` indicating that it was called for a partial shares decrease, say `isPartialRedeem`, and do `addressShares[nftOwner] -= _stakeShares` when `isPartialRedeem == true`.

Another option is to do bigger refactoring, making stakeShares and addressShares always change simultaneously.


