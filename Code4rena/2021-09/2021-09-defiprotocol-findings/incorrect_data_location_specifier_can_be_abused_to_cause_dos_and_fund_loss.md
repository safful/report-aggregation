## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Incorrect data location specifier can be abused to cause DoS and fund loss](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/168) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The withdrawBounty() loops through the _bounties array looking for active bounties and transferring amounts from active ones. However, the data location specifier used for bounty is memory which makes a copy of the _bounties array member instead of a reference. So when bounty.active is set to false, this is changing only the memory copy and not the array element of the storage variable. This results in bounties never being set to inactive, keeping them always active forever and every withdrawBounty() will attempt to transfer bounty amount from the Auction contract to the msg.sender.

Therefore, while the transfer will work the first time, subsequent attempts to claim this bounty will revert on transfer (because the Auction contract will not have required amount of bounty tokens) causing withdrawBounty() to always revert and therefore preventing settling of any auction. 

A malicious attacker can add a tiny bounty on any/every Auction contract to prevent any reindexing on that contract to happen because it will always revert on auction settling. This can be used to cause DoS on any auctionBonder so as to make them lose their bondAmount because their bonded auction cannot be settled.

## Proof of Concept

https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Auction.sol#L143

https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Auction.sol#L143-L147

https://docs.soliditylang.org/en/v0.8.7/types.html#data-location-and-assignment-behaviour

## Tools Used
Manual Analysis

## Recommended Mitigation Steps
Recommend changing storage specifier of bounty to "storage" instead of “memory".

