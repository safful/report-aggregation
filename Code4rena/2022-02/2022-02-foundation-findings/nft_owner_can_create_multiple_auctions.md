## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [NFT owner can create multiple auctions](https://github.com/code-423n4/2022-02-foundation-findings/issues/23) 

# Lines of code

https://github.com/code-423n4/2022-02-foundation/blob/4d8c8931baffae31c7506872bf1100e1598f2754/contracts/mixins/NFTMarketReserveAuction.sol#L325-L349
https://github.com/code-423n4/2022-02-foundation/blob/4d8c8931baffae31c7506872bf1100e1598f2754/contracts/mixins/NFTMarketReserveAuction.sol#L596-L599


# Vulnerability details

# Impact
NFT owner can permanently lock funds of bidders.


# Proof of concept

Alice (the attacker) calls `createReserveAuction`, and creates one like normal. let this be auction id 1.

Alice calls `createReserveAuction` again, before any user has placed a bid (this is easy to guarantee with a deployed attacker contract). We'd expect that Alice wouldn't be able to create another auction, but she can, because `_transferToEscrow` doesn't revert if there's an existing auction. let this be Auction id 2.

Since `nftContractToTokenIdToAuctionId[nftContract][tokenId]` will contain auction id 2, all bidders will see that auction as the one to bid on (unless they inspect contract events or data manually).

Alice can now cancel auction id 1, then cancel auction id 2, locking up the funds of the last bidder on auction id 2 forever.

# Mitigation
Prevent NFT owners from creating multiple auctions

