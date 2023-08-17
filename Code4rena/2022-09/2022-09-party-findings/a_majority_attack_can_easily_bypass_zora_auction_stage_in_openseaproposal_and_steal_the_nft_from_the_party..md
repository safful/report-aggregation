## Tags

- bug
- 3 (High Risk)
- high quality report
- resolved
- sponsor confirmed
- old-submission-method

# [A majority attack can easily bypass Zora auction stage in OpenseaProposal and steal the NFT from the party.](https://github.com/code-423n4/2022-09-party-findings/issues/264) 

# Lines of code

https://github.com/PartyDAO/party-contracts-c4/blob/3896577b8f0fa16cba129dc2867aba786b730c1b/contracts/proposals/ListOnZoraProposal.sol#L176-L183


# Vulnerability details

## Description
The PartyGovernance system has many defenses in place to protect against a majority holder stealing the NFT. One of the main protections is that before listing the NFT on Opensea for a proposal-supplied price, it must first try to be auctioned off on Zora. To move from Zora stage to Opensea stage, _settleZoraAuction() is called when executing ListedOnZora step in ListOnOpenseaProposal.sol. If the function returns false, the next step is executed which lists the item on Opensea. It is assumed that if majority attack proposal reaches this stage, it can steal the NFT for free, because it can list the item for negligible price and immediately purchase it from a contract that executes the Opensea proposal. 

Indeed, attacker can always make settleZoraAuction() return false. Looking at  the code:
```
try ZORA.endAuction(auctionId) {
            // Check whether auction cancelled due to a failed transfer during
            // settlement by seeing if we now possess the NFT.
            if (token.safeOwnerOf(tokenId) == address(this)) {
                emit ZoraAuctionFailed(auctionId);
                return false;
            }
        } catch (bytes memory errData) {
```
As the comment already hints, an auction can be cancelled if the NFT transfer to the bidder fails. This is the relevant AuctionHouse code (endAuction):
```
{
            // transfer the token to the winner and pay out the participants below
            try IERC721(auctions[auctionId].tokenContract).safeTransferFrom(address(this), auctions[auctionId].bidder, auctions[auctionId].tokenId) {} catch {
                _handleOutgoingBid(auctions[auctionId].bidder, auctions[auctionId].amount, auctions[auctionId].auctionCurrency);
                _cancelAuction(auctionId);
                return;
 }
```
As most NFTs inherit from OpenZeppelin's ERC721.sol code, safeTransferFrom will run:
```
    function _safeTransfer(
        address from,
        address to,
        uint256 tokenId,
        bytes memory data
    ) internal virtual {
        _transfer(from, to, tokenId);
        require(_checkOnERC721Received(from, to, tokenId, data), "ERC721: transfer to non ERC721Receiver implementer");
    }
```
So, attacker can bid a very high amount on the NFT to ensure it is the winning bid. When AuctionHouse tries to send the NFT to attacker, the safeTransferFrom will fail because attack will not implement an ERC721Receiver. This will force the AuctionHouse to return the bid amount to the bidder and cancel the auction. Importantly, it will lead to a graceful return from endAuction(), which will make settleZoraAuction() return false and progress to the OpenSea stage.

## Impact
A majority attack can easily bypass Zora auction stage and steal the NFT from the party.

## Proof of Concept
1. Pass a ListOnOpenseaProposal with a tiny list price and execute it
2. Create an attacker contract which bids on the NFT an overpriced amount, but does not implement ERC721Receiver. Call its bid() function
3. Wait for the auction to end ( timeout after the bid() call)
4. Create a contract with a function which calls execute() on the proposal and immediately buys the item on Seaport. Call the attack function.

## Tools Used
Manual audit.

## Recommended Mitigation Steps
_settleZoraAuction is called from both ListOnZoraProposal and ListOnOpenseaProposal. If the auction was cancelled due to a failed transfer, as is described in the comment, we would like to handle it differently for each proposal type. For ListOnZoraProposal, it should indeed return false, in order to finish executing the proposal and not to hang the engine. For ListOnOpenseaProposal, the desired behavior is to *revert* in the case of a failed transfer. This is because the next stage is risky and defense against the mentioned attack is required. Therefore, pass a revertOnFail flag to _settleZoraAuction, which will be used like so:
```
// Check whether auction cancelled due to a failed transfer during
// settlement by seeing if we now possess the NFT.
if (token.safeOwnerOf(tokenId) == address(this)) {
	if (revertOnFail) {
		revert("Zora auction failed because of transfer to bidder")
	}
           emit ZoraAuctionFailed(auctionId);
           return false;
}
```


