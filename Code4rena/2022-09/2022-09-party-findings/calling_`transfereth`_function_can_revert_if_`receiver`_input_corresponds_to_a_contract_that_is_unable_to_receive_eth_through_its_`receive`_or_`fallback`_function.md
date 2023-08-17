## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Calling `transferEth` function can revert if `receiver` input corresponds to a contract that is unable to receive ETH through its `receive` or `fallback` function](https://github.com/code-423n4/2022-09-party-findings/issues/212) 

# Lines of code

https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/utils/LibAddress.sol#L8-L15
https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/crowdfund/Crowdfund.sol#L444-L489
https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/distribution/TokenDistributor.sol#L371-L388


# Vulnerability details

## Impact
The following `transferEth` function is called when calling the `_burn` or `_transfer` function below. If the `receiver` input for the `transferEth` function corresponds to a contract, it is possible that the receiver contract does not, intentionally or unintentionally, implement the `receive` or `fallback` function in a way that supports receiving ETH or that calling the receiver contract's `receive` or `fallback` function executes complicated logics that cost much gas, which could cause calling `transferEth` to revert. For example, when calling `transferEth` reverts, calling `_burn` also reverts; this means that the receiver contract would not be able to get the voting power and receive the extra contribution it made after the crowdfunding finishes; yet, the receiver contract deserves these voting power and contribution refund. Hence, the receiver contract loses valuables that it deserves, which is unfair to the users who controls it.

https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/utils/LibAddress.sol#L8-L15
```solidity
    function transferEth(address payable receiver, uint256 amount)
        internal
    {
        (bool s, bytes memory r) = receiver.call{value: amount}("");
        if (!s) {
            revert EthTransferFailed(receiver, r);
        }
    }
```

https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/crowdfund/Crowdfund.sol#L444-L489
```solidity
    function _burn(address payable contributor, CrowdfundLifecycle lc, Party party_)
        private
    {
        // If the CF has won, a party must have been created prior.
        if (lc == CrowdfundLifecycle.Won) {
            if (party_ == Party(payable(0))) {
                revert NoPartyError();
            }
        } else if (lc != CrowdfundLifecycle.Lost) {
            // Otherwise it must have lost.
            revert WrongLifecycleError(lc);
        }
        // Split recipient can burn even if they don't have a token.
        if (contributor == splitRecipient) {
            if (_splitRecipientHasBurned) {
                revert SplitRecipientAlreadyBurnedError();
            }
            _splitRecipientHasBurned = true;
        }
        // Revert if already burned or does not exist.
        if (splitRecipient != contributor || _doesTokenExistFor(contributor)) {
            CrowdfundNFT._burn(contributor);
        }
        // Compute the contributions used and owed to the contributor, along
        // with the voting power they'll have in the governance stage.
        (uint256 ethUsed, uint256 ethOwed, uint256 votingPower) =
            _getFinalContribution(contributor);
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
        // Refund any ETH owed back to the contributor.
        contributor.transferEth(ethOwed);
        emit Burned(contributor, ethUsed, ethOwed, votingPower);
    }
```

https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/distribution/TokenDistributor.sol#L371-L388
```solidity
    function _transfer(
        TokenType tokenType,
        address token,
        address payable recipient,
        uint256 amount
    )
        private
    {
        bytes32 balanceId = _getBalanceId(tokenType, token);
        // Reduce stored token balance.
        _storedBalances[balanceId] -= amount;
        if (tokenType == TokenType.Native) {
            recipient.transferEth(amount);
        } else {
            assert(tokenType == TokenType.Erc20);
            IERC20(token).compatTransfer(recipient, amount);
        }
    }
```

## Proof of Concept
Please add the following `error` and append the test in `sol-tests\crowdfund\BuyCrowdfund.t.sol`. This test will pass to demonstrate the described scenario.

```solidity
    error EthTransferFailed(address receiver, bytes errData);

    function testContributorContractFailsToReceiveETH() public {
        uint256 tokenId = erc721Vault.mint();
        BuyCrowdfund pb = _createCrowdfund(tokenId, 0);

        // This contract is used to simulate a contract that does not implement the receive or fallback function for the purpose of receiving ETH.
        address payable contributorContract = payable(address(this));
        vm.deal(contributorContract, 1e18);

        address delegate = _randomAddress();

        // contributorContract contributes 1e18.
        vm.prank(contributorContract);
        pb.contribute{ value: 1e18 }(delegate, "");

        // The price of the NFT of interest is 0.5e18.
        Party party_ = pb.buy(
            payable(address(erc721Vault)),
            0.5e18,
            abi.encodeCall(erc721Vault.claim, (tokenId)),
            defaultGovernanceOpts
        );

        // After calling the buy function, the party is created with the NFT.
        assertEq(address(party), address(party_));
        assertTrue(pb.getCrowdfundLifecycle() == Crowdfund.CrowdfundLifecycle.Won);
        assertEq(pb.settledPrice(), 0.5e18);
        assertEq(pb.totalContributions(), 1e18);
        assertEq(address(pb).balance, 1e18 - 0.5e18);

        // Calling the burn function reverts because contributorContract cannot receive ETH through the receive or fallback function
        vm.expectRevert(abi.encodeWithSelector(
            EthTransferFailed.selector,
            contributorContract,
            ""
        ));
        pb.burn(contributorContract);

        // contributorContract does not receive 0.5e18 back from the BuyCrowdfund contract.
        assertEq(contributorContract.balance, 0);
    }
```

## Tools Used
VSCode

## Recommended Mitigation Steps
When calling the `transferEth` function, if the receiver contract is unable to receive ETH through its `receive` or `fallback` function, WETH can be used to deposit the corresponding ETH amount, and the deposited amount can be transferred to the receiver contract.