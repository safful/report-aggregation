## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- edited-by-warden

# [The quorum votes calculations don't take into account burned tokens](https://github.com/code-423n4/2022-09-nouns-builder-findings/issues/423) 

# Lines of code

https://github.com/code-423n4/2022-09-nouns-builder/blob/main/src/governance/governor/Governor.sol#L475
https://github.com/code-423n4/2022-09-nouns-builder/blob/main/src/governance/governor/Governor.sol#L524


# Vulnerability details

Because the following happens:

1. Burned tokens votes are effectively deleted in `token._moveDelegateVotes()` when called by `token.burn()`
2. When an auction gets settled without bidders the function burns the token by calling `token.burn()`
3. When `_createAuction()` is called an amount of tokens >= 1 is minted, of which 1 is kept in the auction contract
4. The functions `governor.proposalThreshold()` and `governor.quorum()` both depend on `token.totalSupply()` for their calculations.

We can derive that the protocol calculates the `quorumVotes` taking into account burned tokens and tokens held in the auction contract, which don't have any actual voting power. In other words the actual `quorumThresholdBps` is equal or higher than the setted `quorumThresholdBps`.

## Impact
The worse case scenario that can happen is that the quorum gets so high that a proposal cannot pass even if everybody voted and everybody voted `for`, potentially locking funds into the contract.

We can define:

1. `assumedVotingPower` = `token.totalSupply()`
2. `realVotingPower` = `token.totalSupply() - amountOfTokensBurned`
3. `ΔVotingPower` = `amountOfTokensBurned`


This is the case if:

```
realVotingPower at proposal.voteEnd < quorum at proposal.timeCreated
```
which is the same as

```
realVotingPower < (assumedVotingPower * settings.quorumThresholdBps) / 10_000
```
and rearranging in terms of `settings.quorumThresholdBps` we have:

```
settings.quorumThresholdBps > 10_000 * realVotingPower/assumedVotingPower
```

Knowing that:

1. The possible range of values for `10_000 * realVotingPower/assumedVotingPower` is from `1` to `10000`. If `realVotingPower = 0` this model doesn't make sense in the first place. 

2. The possible range of values of `settings.quorumThresholdBps` is from `1` to `2^16 - 1`. The protocol allows for `settings.quorumThresholdBps` to be `0`, in which case it means that the actual quorum is `0`; a context in which this model doesn't make sense. There's another catch that restricts this boundaries, if `settings.quorumThresholdBps * token.totalSupply()` < `10_000` the output of `governance.quorum()` would be `0`. 

Many combinations of values in the ranges described above render this disequation true, note, however, that this describes the workings in a mathematical settings and it doesnt hold true for every case in a real setting because of roundings and approximations.

We can intuitevely notice that when `realVotingPower/assumedVotingPower` is very low, which is the case of a DAO with few tokens burned, the chances of the disequation being true are slim and when it's high the chances of the disequation being true become higher. The opposite is true for `settings.quorumThresholdBps`.

This might lock funds in DAOs with a lot of unsold auctions who have a low `settings.quorumThresholdBps`.

At early stages this is mitigated by the fact that for every possible token burned some tokens are minted to the founders, but when the vest expires this mitigation is not in place anymore.

## Proof of concept

I wrote a test that's expected to revert a `proposal.queue()` even if all possible votes available are cast in favor.

The test comes with two parameters to set: `auctionsToRun` and `tokensToBidder`. The test runs `auctionsToRun` auctions, of which the first `tokensToBidder` are bidded upon and the rest are not. Then:
1. Creates a proposal
2. Cast all possible votes in favor
3. Tries to queue a proposal
4. Reverts

The default parameters are set to `auctionsToRun = 130` and `tokensToBidder = 10`. Also `quorumThresholdBps = 1000`. This test results in `121 tokens burned` and `133 token minted`. It's quite an unrealistic scenario, but it can get more real if `quorumThresholdBps` is setted lower. Keep in mind that this is the case in which everybody shows up to vote and averybody votes for.

### Test code
The test can be pasted inside `Gov.t.sol` and then run with:

`test -m test_RevertQueueProposalWithEverybodyInFavour`

```javascript
function test_RevertQueueProposalWithEverybodyInFavour() public {
    //Number of auctions to run
    uint256 auctionsToRun = 130;

    //Amount of tokens to bid up
    uint256 tokensToBidder = 10;

    address bidder1 = vm.addr(0xB1);
    vm.deal(founder, 10000 ether);
    vm.deal(bidder1, 10000 ether);

    //Start the first auction
    vm.prank(founder);
    auction.unpause();

    //Simulates an `auctionsToRun` amount of auctions in which the first `tokensForBidder` tokens
    //are minted and then every auction ends with no bidders.
    uint256 amountOfBurnedTokens;
    for (uint256 i = 1; i < auctionsToRun + 1; ++i) {
        if (i < tokensToBidder) {
            uint256 id = token.totalSupply() - 1;
            vm.prank(bidder1);
            auction.createBid{ value: 0.15 ether }(id);
        } else {
            amountOfBurnedTokens++;
        }

        vm.warp(block.timestamp + auction.duration() + 1);
        auction.settleCurrentAndCreateNewAuction();
    }

    uint256 founderVotes = token.getVotes(founder);
    uint256 founder2Votes = token.getVotes(founder2);
    uint256 bidder1Votes = token.getVotes(bidder1);
    uint256 auctionVotes = token.getVotes(address(auction));

    uint256 realVotingPower = founderVotes + founder2Votes + bidder1Votes;
    uint256 assumedVotingPower = token.totalSupply();

    assertEq(realVotingPower, assumedVotingPower - amountOfBurnedTokens - auctionVotes);

    //Create mock proposal
    (address[] memory targets, uint256[] memory values, bytes[] memory calldatas) = mockProposal();
    vm.prank(bidder1);
    bytes32 proposalId = governor.propose(targets, values, calldatas, "");

    emit log_string("Amount of tokens minted: ");
    emit log_uint(token.totalSupply());

    emit log_string("Amount of tokens burned:");
    emit log_uint(amountOfBurnedTokens);

    emit log_string("---------");

    emit log_string("The real quorumThresholdBps is: ");
    uint256 realquorumThresholdBps = (governor.getProposal(proposalId).quorumVotes * 10_000) / realVotingPower;
    emit log_uint(realquorumThresholdBps);

    emit log_string("The assumed quorumThresholdBps is:");
    uint256 assumedquorumThresholdBps = (governor.getProposal(proposalId).quorumVotes * 10_000) / token.totalSupply();
    emit log_uint(assumedquorumThresholdBps);

    emit log_string("---------");

    vm.warp(governor.getProposal(proposalId).voteStart);

    //Everybody cast a `for` vote
    vm.prank(founder);
    governor.castVote(proposalId, 1);

    vm.prank(founder2);
    governor.castVote(proposalId, 1);

    vm.prank(bidder1);
    governor.castVote(proposalId, 1);

    emit log_string("The amount of votes necessary for this proposal to pass is:");
    emit log_uint(governor.getProposal(proposalId).quorumVotes);

    emit log_string("The amount of for votes in the proposal:");
    emit log_uint(governor.getProposal(proposalId).forVotes);

    //Proposal still doesn't pass
    vm.warp((governor.getProposal(proposalId).voteEnd));
    vm.expectRevert(abi.encodeWithSignature("PROPOSAL_UNSUCCESSFUL()"));
    governor.queue(proposalId);
}
```

## Tools Used
Forge

## Recommended Mitigation Steps
Either one of this 2 options is viable:
1. Decrease `token.totalSupply()` whenever a token gets burned. This might not be expected behaviour from the point of view of external protocols.
2. Adjust the calculations in `proposal.quorum()` and `governor.proposalThreshold()` in such a way that they take into account the burned tokens and the tokens currently held by the auction contract.