## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- high quality report
- resolved
- sponsor confirmed
- old-submission-method

# [Previously nominated delegate can reset the delegation](https://github.com/code-423n4/2022-09-party-findings/issues/361) 

# Lines of code

https://github.com/PartyDAO/party-contracts-c4/blob/3896577b8f0fa16cba129dc2867aba786b730c1b/contracts/crowdfund/Crowdfund.sol#L167-L171


# Vulnerability details

   

burn() allows for previously recorded delegate to set himself to be contributor's delegate even if another one was already chosen.

This can be quite material as owner choice for the whole voting power is being reset this way to favor the old delegate.

## Proof of Concept

_burn() can be invoked by anyone on the behalf of any `contributor`:

https://github.com/PartyDAO/party-contracts-c4/blob/3896577b8f0fa16cba129dc2867aba786b730c1b/contracts/crowdfund/Crowdfund.sol#L167-L171

```solidity
    function burn(address payable contributor)
        public
    {
        return _burn(contributor, getCrowdfundLifecycle(), party);
    }
```

It mints the governance NFT for the contributor whenever he has voting power:

https://github.com/PartyDAO/party-contracts-c4/blob/3896577b8f0fa16cba129dc2867aba786b730c1b/contracts/crowdfund/Crowdfund.sol#L471-L485

```solidity
        if (votingPower > 0) {
            // Get the address to delegate voting power to. If null, delegate to self.
            address delegate = delegationsByContributor[contributor];
            if (delegate == address(0)) {
                // Delegate can be unset for the split recipient if they never
                // contribute. Self-delegate if this occurs.
                delegate = contributor;
            }
            // Mint governance NFT for the contributor.
            party_.mint(
                contributor,
                votingPower,
                delegate
            );
        }
```

Now mint() calls _adjustVotingPower() with a new delegate, redirecting all the intristic power, not just one for that id, ignoring the delegation the `owner` might already have:

https://github.com/PartyDAO/party-contracts-c4/blob/3896577b8f0fa16cba129dc2867aba786b730c1b/contracts/party/PartyGovernanceNFT.sol#L120-L133

```solidity
    function mint(
        address owner,
        uint256 votingPower,
        address delegate
    )
        external
        onlyMinter
        onlyDelegateCall
    {
        uint256 tokenId = ++tokenCount;
        votingPowerByTokenId[tokenId] = votingPower;
        _adjustVotingPower(owner, votingPower.safeCastUint256ToInt192(), delegate);
        _mint(owner, tokenId);
    }
```

I.e. Bob the contributor can take part in the crowdfunding with contribute() with small `0.01 ETH` stake, stating Mike as the delegate of his choice with `contribute(Mike, ...)`:

https://github.com/PartyDAO/party-contracts-c4/blob/3896577b8f0fa16cba129dc2867aba786b730c1b/contracts/crowdfund/Crowdfund.sol#L189-L208

```solidity
    /// @param delegate The address to delegate to for the governance phase.
    /// @param gateData Data to pass to the gatekeeper to prove eligibility.
    function contribute(address delegate, bytes memory gateData)
        public
        payable
    {
        _contribute(
            msg.sender,
            msg.value.safeCastUint256ToUint96(),
            delegate,
            // We cannot use `address(this).balance - msg.value` as the previous
            // total contributions in case someone forces (suicides) ETH into this
            // contract. This wouldn't be such a big deal for open crowdfunds
            // but private ones (protected by a gatekeeper) could be griefed
            // because it would ultimately result in governance power that
            // is unattributed/unclaimable, meaning that party will never be
            // able to reach 100% consensus.
            totalContributions,
            gateData
        );
```

Then crowdfund was a success, party was created, and Melany, who also participated, per off-chain arrangement has transferred to Bob a `tokenId` with big voting power (say it is `100 ETH` and the majority of voting power):

https://github.com/PartyDAO/party-contracts-c4/blob/3896577b8f0fa16cba129dc2867aba786b730c1b/contracts/party/PartyGovernanceNFT.sol#L146-L155

```solidity
    /// @inheritdoc ERC721
    function safeTransferFrom(address owner, address to, uint256 tokenId)
        public
        override
        onlyDelegateCall
    {
        // Transfer voting along with token.
        _transferVotingPower(owner, to, votingPowerByTokenId[tokenId]);
        super.safeTransferFrom(owner, to, tokenId);
    }
```

https://github.com/PartyDAO/party-contracts-c4/blob/3896577b8f0fa16cba129dc2867aba786b730c1b/contracts/party/PartyGovernance.sol#L879-L887

```solidity
    // Transfers some voting power of `from` to `to`. The total voting power of
    // their respective delegates will be updated as well.
    function _transferVotingPower(address from, address to, uint256 power)
        internal
    {
        int192 powerI192 = power.safeCastUint256ToInt192();
        _adjustVotingPower(from, -powerI192, address(0));
        _adjustVotingPower(to, powerI192, address(0));
    }
```

Bob don't care about his early small contribution and focuses on managing the one that Melany transferred instead as he simply don't need the voting power from the initial `0.01 ETH` contribution anymore.

The actual delegate for Bob at the moment is Linda, while his business with Mike is over. So Bob sets her address there, calling `delegateVotingPower(Linda)`:

https://github.com/PartyDAO/party-contracts-c4/blob/3896577b8f0fa16cba129dc2867aba786b730c1b/contracts/party/PartyGovernance.sol#L448-L454

```solidity
    /// @notice Pledge your intrinsic voting power to a new delegate, removing it from
    ///         the old one (if any).
    /// @param delegate The address to delegating voting power to.
    function delegateVotingPower(address delegate) external onlyDelegateCall {
        _adjustVotingPower(msg.sender, 0, delegate);
        emit VotingPowerDelegated(msg.sender, delegate);
    }
```

Now, Mike can unilaterally delegate to himself the whole voting power with `burn(Bob)` as mint() just resets the delegation with the previously recorded value with `_adjustVotingPower(owner, votingPower.safeCastUint256ToInt192(), delegate)`.

## Recommended Mitigation Steps

The issue is that mint() always assumes that it is the first operation for the `owner`, which might not always be the case.

Consider not changing the delegate on `mint` if one is set already:

https://github.com/PartyDAO/party-contracts-c4/blob/3896577b8f0fa16cba129dc2867aba786b730c1b/contracts/party/PartyGovernanceNFT.sol#L120-L133

```solidity
    function mint(
        address owner,
        uint256 votingPower,
        address delegate
    )
        external
        onlyMinter
        onlyDelegateCall
    {
        uint256 tokenId = ++tokenCount;
        votingPowerByTokenId[tokenId] = votingPower;
+       address actualDelegate = <get_current_delegate>;
+       if (actualDelegate == address(0)) actualDelegate = delegate;
-       _adjustVotingPower(owner, votingPower.safeCastUint256ToInt192(), delegate);
+       _adjustVotingPower(owner, votingPower.safeCastUint256ToInt192(), actualDelegate);
        _mint(owner, tokenId);
    }
```

More complicated version might be the one with tracking the most recent request via contribute()/delegateVotingPower() calls timestamps. Here we assume that the delegateVotingPower() holds more information as in the majority of practical cases it occurs after initial contribute() and it is a direct voluntary call from the owner. 

