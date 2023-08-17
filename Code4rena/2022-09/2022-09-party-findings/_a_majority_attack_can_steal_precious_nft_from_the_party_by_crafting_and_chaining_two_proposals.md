## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed
- old-submission-method

# [ A majority attack can steal precious NFT from the party by crafting and chaining two proposals](https://github.com/code-423n4/2022-09-party-findings/issues/277) 

# Lines of code

https://github.com/PartyDAO/party-contracts-c4/blob/3896577b8f0fa16cba129dc2867aba786b730c1b/contracts/proposals/ProposalExecutionEngine.sol#L116
https://github.com/PartyDAO/party-contracts-c4/blob/3896577b8f0fa16cba129dc2867aba786b730c1b/contracts/proposals/FractionalizeProposal.sol#L54-L62


# Vulnerability details

## Description
The PartyGovernance system has many defenses in place to protect against a majority holder stealing the NFT. Majority cannot exfiltrate the ETH gained from selling precious NFT via any proposal, and it's impossible to sell NFT for any asset except ETH. If the party were to be compensated via an ERC20 token, majority could pass an ArbitraryCallsProposal to transfer these tokens to an attacker wallet. Unfortunately, FractionalizeProposal is vulnerable to this attack. Attacker/s could pass two proposals and wait for them to be ready for execution. Firstly, a FractionalizeProposal to fractionalize the NFT and mint totalVotingPower amount of ERC20 tokens of the created vault. Secondly, an ArbitraryCallsProposal to transfer the entire ERC20 token supply to an attacker address. At this point, attacker can call vault.redeem() to burn the outstanding token supply and receive the NFT back.

## Impact
A 51% majority could steal the precious NFT from the party and leave it empty.

## Proof of Concept
The only non-trivial component of this attack is that the created vault, whose tokens we wish to transfer out, has an undetermined address until VAULT_FACTORY.mint() is called, which creates it. The opcode which creates the vault contract is CREATE, which calculates the address with ```keccak256(VAULT_FACTORY, nonce)```. Nonce will keep changing while new, unrelated NFTs are fractionalized. The attack needs to prepare both FractionalizedProposal and ArbitraryCallsProposal ahead of time, so that they could be chained immediately, meaning there would be no time for other members to call distribute() on the party, which would store the fractionalized tokens safely in the distributor.
In order to solve this chicken and the egg problem, we will use a technique taken from traditional low-level exploitation called heap feng shui. 

Firstly, calculate off-chain, the rate new NFTs are fractionalized, and multiple by a safety factor (like 1.2X), and multiply again by the proposal execution delay. This number, added to the current VAULT_FACTORY nonce, will be our target_nonce. Calculate ```target_vault = keccak256(VAULT_FACTORY, target_nonce)```, ```before_target_vault = keccak256(VAULT_FACTORY, target_nonce-1)```

Firstly, we will create a contract which has an attack function that:
1. Loop while before_target_vault != created_vault:
	• Mint new dummy attacker_NFT
	• created_vault = VAULT_FACTORY.mint(attacker_NFT…)
2. Call execute() on the FractionalizedProposal  // We will feed the execute() parameters to the contract in a separate contract setter. Note that this is guaranteed to create target_vault on the correct address.
3. Call execute() on the ArbitraryCallsProposal

Then, we propose the two proposals:
1. Propose a FractionalizedProposal, with any list price and the precious NFT as parameter
2. Propose an ArbitraryCallsProposal, with target = target_vault, data = transfer(ATTACKER, totalVotingPower)

Then, we set the execute() parameters passed in step 2 and 3 of the attack contract using the proposalID allocated for them. 

Then, we wait for execution delay to finish.

Finally, run the attack() function prepared earlier. This will increment the VAULT_FACTORY nonce until it is the one we count on during the ArbitraryCallsProposal. Pass enough gas to be able to burn enough nonces.

At this point, attacker has all the vault tokens, so he may call vault.redeem() and receive the precious NFT.

## Tools Used
Manual audit.

## Recommended Mitigation Steps
1. Enforce a minimum cooldown between proposals. This will mitigate additional weaknesses of the proposal structure. Here, this will give users the opportunity to call distribute() to put the vault tokens safe in distributor.
2. A specific fix here would be to call distribute() at the end of FractionalizeProposal so that there is no window to steal the funds.

