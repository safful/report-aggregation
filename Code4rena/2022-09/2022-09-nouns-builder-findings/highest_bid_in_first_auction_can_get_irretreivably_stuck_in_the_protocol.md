## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Highest bid in first auction can get irretreivably stuck in the protocol](https://github.com/code-423n4/2022-09-nouns-builder-findings/issues/376) 

# Lines of code

https://github.com/code-423n4/2022-09-nouns-builder/blob/7e9fddbbacdd7d7812e912a369cfd862ee67dc03/src/auction/Auction.sol#L248-L254


# Vulnerability details

## Impact
If the first auction is paused and unpaused in a protocol deployed with no founder fees, the highest bid (as well as the first NFT), will get stuck in the protocol with no ability to retrieve either of them.

## Proof of Concept
In a protocol with founder ownership percentage set to 0, the first tokenId put to auction is #0.

If the first auction in such a protocol is paused and unpaused, the check for `if (auction.tokenId == 0)` will pass and `_createAuction()` will automatically be called, minting the next token and starting a new auction based on token #1.

The result is that `highestBid` and `highestBidder` are reset, the first auction is never settled, and the highest bid (as well as NFT #0) will remain stuck in the platform.

The following test confirms this finding:

```solidity
function test_PauseAndUnpauseInFirstAuction() public {
    address bidder1 = vm.addr(0xB1);
    address bidder2 = vm.addr(0xB2);

    vm.deal(bidder1, 100 ether);
    vm.deal(bidder2, 100 ether);

    console.log("Deploying with no founder pct...");
    deployMockWithEmptyFounders();

    console.log("Unpausing...");
    vm.prank(founder);
    auction.unpause();

    console.log("Bidder makes initial bid.");
    vm.prank(bidder1);
    auction.createBid{ value: 1 ether }(0);
    (uint256 tokenId_, uint256 highestBid_, address highestBidder_,,,) = auction.auction();
    console.log("Currently bidding for ID ", tokenId_);
    console.log("Highest Bid: ", highestBid_, ". Bidder: ", highestBidder_);
    console.log("Contract Balance: ", address(auction).balance);
    console.log("--------");

    console.log("Pausing and unpausing auction house...");
    vm.startPrank(address(treasury));
    auction.pause();
    auction.unpause();
    vm.stopPrank();

    console.log("Bidder makes new bid.");
    vm.prank(bidder2);
    auction.createBid{ value: 0.5 ether }(1);
    (uint256 tokenId2_, uint256 highestBid2_, address highestBidder2_,,,) = auction.auction();
    console.log("Currently bidding for ID ", tokenId2_);
    console.log("Highest Bid: ", highestBid2_, ". Bidder: ", highestBidder2_);
    console.log("Contract Balance: ", address(auction).balance);
```

## Tools Used

Manual Review, Foundry

## Recommended Mitigation Steps

Remove the block in `unpause()` that transfers ownership and creates an auction if `auction.tokenId == 0` and trigger those actions manually in the deployment flow.