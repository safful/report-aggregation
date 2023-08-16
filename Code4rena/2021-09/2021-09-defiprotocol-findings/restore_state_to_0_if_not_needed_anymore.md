## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Restore state to 0 if not needed anymore](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/40) 

# Handle

kenzo


# Vulnerability details

In some places where data is discarded such as ```bondBurn```, part of the data is set to 0 (```auctionBonder```), and other parts are not (```bondTimestamp```).
Setting unnecessary data back to 0 will save gas.

## Impact
Almost 2000 gas saved for each variable reset.
In some places, like ```createBasket``` (which only needs to save the proposal's "basket" field after creating the basket), this can save almost 15000 gas.

## Proof of Concept
Places where data is not reset:
Factory's createBasket (set all _proposals[idNumber]'s fields to be 0 except basket)
https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Factory.sol#L112
Basket's changePublisher: (set pendingPublisher.block = 0)
https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Basket.sol#L141
Basket's changeLicenseFee: (set pendingLicenseFee.block = 0)
https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Basket.sol#L159
Basket's setNewWeights and deleteNewIndex: (set pendingWeights.tokens and pendingWeights.weights to empty arrays)
https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Basket.sol#L200
https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Basket.sol#L212
Auction's killAuction: (set auctionStart = 0)
https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Auction.sol#L44
Auction's settleAuction: (set bondTimestamp, auctionBonder = 0)
https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Auction.sol#L107
Auction's bondBurn: (set bondTimestamp = 0)
https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Auction.sol#L120
Auction's withdrawBounty: (set bounty.token, bounty.token = 0)
https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Auction.sol#L148

## Tools Used
Manual analysis, hardhat.

## Recommended Mitigation Steps
Detailed above.

