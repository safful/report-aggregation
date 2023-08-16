## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Unconventional log emittance confuses Etherscan](https://github.com/code-423n4/2021-11-unlock-findings/issues/90) 

# Handle

kenzo


# Vulnerability details

Unlock doesn't follow standard ERC721 log emittance.
This leads to wrong display values regarding to the lock NFT on Etherscan.

## Impact
Etherscan does not show txs correctly, does not count token holders correctly in token page, does not count tokens correctly in user page.

## Proof of Concept
A scenario:
- Create a new lock
- User 1 mints 1 token
- User 1 uses `shareKey` and transfers some amount to User 2
At this point Etherscan will show that 3 transfers have been made, under user 2's address page user 2 has 2 keys , and under lock's holders tab user 2 has 2 keys. All this is obviously wrong.
This is probably because the transfer event is emitted twice during shareKey: [here](https://github.com/code-423n4/2021-11-unlock/blob/main/smart-contracts/contracts/mixins/MixinTransfer.sol#L87) and [here](https://github.com/code-423n4/2021-11-unlock/blob/main/smart-contracts/contracts/mixins/MixinTransfer.sol#L100).

Additionally, if user 1 now calls `Cancel And Refund`, user 1 will still have a key under his tokens in his account, and the lock's token page will still list him as a holder, and the transaction won't get shown in Etherscan's token transfers (unlike contract transactions). Probably because it has not emitted any burn event. It just emits a [CancelKey event](https://github.com/code-423n4/2021-11-unlock/blob/main/smart-contracts/contracts/mixins/MixinRefunds.sol#L111).

## Recommended Mitigation Steps
You can align the logs emittance to match regular ERC721 logs if you'd like Etherscan to show correct amounts. It might get confusing to keep it like this.

