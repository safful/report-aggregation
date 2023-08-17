## Tags

- bug
- 2 (Med Risk)
- primary issue
- selected for report
- sponsor confirmed
- fix security (sponsor)
- M-01

# [RewardsPool.sol : It is safe to have the startRewardsCycle with WhenNotPaused modifier](https://github.com/code-423n4/2022-12-gogopool-findings/issues/823) 

# Lines of code

https://github.com/code-423n4/2022-12-gogopool/blob/aec9928d8bdce8a5a4efe45f54c39d4fc7313731/contracts/contract/RewardsPool.sol#L155-L197


# Vulnerability details

## Impact
when the contract is paused , allowing startRewardsCycle would inflate the token value which might not be safe.

Rewards should not be claimed by anyone when all other operations are paused.

I saw that the `witdrawGGP` has this `WhenNotPaused` modifier.

Inflate should not consider the paused duration.

lets say, when the contract is paused for theduration of 2 months, then the dao, protocol, and node validator would enjoy the rewards. This is not good for a health protocol

## Proof of Concept

startRewardsCycle does not have the WhenNotPaused modifier.

	function startRewardsCycle() external {
		if (!canStartRewardsCycle()) {
			revert UnableToStartRewardsCycle();
		}


		emit NewRewardsCycleStarted(getRewardsCycleTotalAmt());


		// Set start of new rewards cycle
		setUint(keccak256("RewardsPool.RewardsCycleStartTime"), block.timestamp);
		increaseRewardsCycleCount();
		// Mint any new tokens from GGP inflation
		// This will always 'mint' (release) new tokens if the rewards cycle length requirement is met
		// 		since inflation is on a 1 day interval and it needs at least one cycle since last calculation
		inflate();


		uint256 multisigClaimContractAllotment = getClaimingContractDistribution("ClaimMultisig");
		uint256 nopClaimContractAllotment = getClaimingContractDistribution("ClaimNodeOp");
		uint256 daoClaimContractAllotment = getClaimingContractDistribution("ClaimProtocolDAO");
		if (daoClaimContractAllotment + nopClaimContractAllotment + multisigClaimContractAllotment > getRewardsCycleTotalAmt()) {
			revert IncorrectRewardsDistribution();
		}


		TokenGGP ggp = TokenGGP(getContractAddress("TokenGGP"));
		Vault vault = Vault(getContractAddress("Vault"));


		if (daoClaimContractAllotment > 0) {
			emit ProtocolDAORewardsTransfered(daoClaimContractAllotment);
			vault.transferToken("ClaimProtocolDAO", ggp, daoClaimContractAllotment);
		}


		if (multisigClaimContractAllotment > 0) {
			emit MultisigRewardsTransfered(multisigClaimContractAllotment);
			distributeMultisigAllotment(multisigClaimContractAllotment, vault, ggp);
		}


		if (nopClaimContractAllotment > 0) {
			emit ClaimNodeOpRewardsTransfered(nopClaimContractAllotment);
			ClaimNodeOp nopClaim = ClaimNodeOp(getContractAddress("ClaimNodeOp"));
			nopClaim.setRewardsCycleTotal(nopClaimContractAllotment);
			vault.transferToken("ClaimNodeOp", ggp, nopClaimContractAllotment);
		}
	}

## Tools Used

Manual review

## Recommended Mitigation Steps

We suggest to use `WhenNotPaused` modifier.